==============================================================
[2017_Inc0gnito] [web] corneria
==============================================================

문제내용
==============================================================

Please.. come..
Service available at: Here
Source code available at: Here


문제 풀이
==============================================================

.. code-block:: php

    <?php 
        define("FLAG","flag{????????????}");

        class info {
            var $i;
            public function __get($name) {
                return $this->$name;
            }
            function infomation($my_level) {
                $i = array_reverse(explode("|",$my_level));
                foreach($i as $hell_buf) {
                    $input_id=explode("=",$hell_buf);
                    if($input_id[0]==='id')
                        $this->i['id']=$input_id[1]; 
                    else if($input_id[0]==='level')
                        $this->i['authLevel']=(int)$input_id[1]; 
                    else if($input_id[0]==='pw') {
                        $this->i['pw']=$input_id[1];
                    }
                }
            }
            function my_password() {
                return $this->i['pw'];
            }
            function input_id() {
                return $this->i['id'];
            }
            function my_level() {
                return $this->i['authLevel'];
            }
            public function __construct() {
                $this->i['id']="default";
                $this->i['authLevel']=(int)0;
                $this->i['pw']=md5("default_passwd");
            }
            function hell_buf($a) {
                $i=rand(0,getrandmax());
                $i=md5((string)$i.$a);
                return $i;
            }
        }

        $i=new info;
        if(isset($_POST['id'])&&isset($_POST['pw'])) {
            $hell_buf=true;
            $input_id=md5($_POST['pw']);
        $my_level=$_POST['id'];
        if(($input_id)==="is_this_really_md5?") {
            $my_password=1;
        } else {
            $my_password=0;
        }
        $infomation=array();
        $infomation['id'] = "id=".$my_level;
        $infomation['level'] = "level=".$my_password;
        $infomation['pw'] = "pw=".$i->hell_buf($input_id);
        $data=implode("|",$infomation);
        $i->infomation($data);
        if($i->my_level()===1) {
            echo FLAG;
        }
    } else {
        $hell_buf=false;
    }
    ?>
    <html><head><title>My auth site</title></head><body><h4>Hello <?php if($hell_buf) echo ", ".$i->input_id(); ?></h4>
    <?php if($hell_buf) {
    echo "Your ID : ".$i->input_id()."<br>Your Level : ".$i->my_level()."<br>Your password : ".$i->my_password()."<br>";
    } else {
    echo "<form method='post' action='".$_SERVER['PHP_SELF']."'>ID : <input type = 'text' name='id' /><br>PW : <input type = 'password' name='pw' /><br><input type='submit' /></form>";
    }
    ?></body></html>


문제에서 PHP 소스코드가 공개되어 있는데, 해당 소스코드를 보면 "|"를 통해 explode를 하는 것을 확인할 수 있다.
그리고 플래그는 level값이 1일 경우 출력되도록 되어 있으므로 id부분에 조건을 만족시키는 level=1값을 입력하면 플래그가 출력된다.
