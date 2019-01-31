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
>또 그누보드엔 각각의 게시판마다 g5_board 테이블에 있는 bo_include_head,bo_include_head 컬럼들의 값을 각각 상단 파일,하단 파일로 include합니다
