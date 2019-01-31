# 그누보드 5.2.9.8.4 RCE
>해당 취약점은 그누보드 5.2.9.8.4 에서 터지는 SQL Injection을 이용한 RCE 취약점입니다.
>
## 취약점 설명
>그누보드 /bbs/move_update.php 29~30,164 라인
<pre><code>$move_bo_table = $_POST['chk_bo_table'][$i];
$move_write_table = $g5['write_prefix'] . $move_bo_table;</code></pre>
>위의 코드를 보면 별다른 검증없이 "$_POST['chk_bo_table'][$i]" 변수의 값을 "$move_write_table" 변수에 넣어주는것을 알 수 있습니다.
<pre><code>sql_query(" update $move_write_table set wr_parent = '$save_parent' where wr_id = '$insert_id' ");</code></pre>
>그리고 위와 같이 sql_query함수를 이용하여 update 하는 SQL을 날려주는데 이때 테이블 clause에서 SQL Injection이 가능하고 update는 다른 테이블 참조 등이 가능합니다.



>그누보드에서 파일을 업로드하는 기능이 있는데 이 파일을 업로드 하면 "g5_board_file" 라는 테이블에 파일 명, 게시판 명, 게시글 번호 등등이 Insert되고 이 데이터를 기반으로 게시글에서는 파일 다운로드, 파일 삭제 등의 작업을 진행합니다.

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


>그누보드에서는 각각의 게시판마다 g5_board 테이블에 있는 bo_include_head,bo_include_head 컬럼들의 값을 각각 상단 파일,하단 파일로 include합니다
