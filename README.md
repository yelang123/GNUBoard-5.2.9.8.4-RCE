# 그누보드 5.2.9.8.4 RCE
>해당 취약점은 그누보드 5.2.9.8.4 에서 터지는 SQL Injection을 이용한 RCE 취약점입니다.
>
## 취약점 설명
>그누보드 /bbs/move_update.php 29~30,164 라인
><pre><code>        $move_bo_table = $_POST['chk_bo_table'][$i];
        $move_write_table = $g5['write_prefix'] . $move_bo_table;</code></pre>
