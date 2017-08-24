---
title: XMLHttpRequest FormData入门
date: 2016-11-06 19:08:05
tags: [HTTP, XMLHttpRequest]
category: HTTP协议
---

> `XMLHttpRequest Level 2`添加了一个新的接口`FormData`。利用`FormData`对象，我们可以通过JavaScript用一些键值对来模拟一系列表单控件，我们还可以使用`XMLHttpRequest`的`send()`方法来异步的提交这个表单。比起普通的ajax，使用`FormData`的最大优点就是我们可以异步上传一个二进制文件。


## 创建FormData对象

利用FormData对象，你可以使用一系列的键值对来模拟一个完整的表单，然后使用XMLHttpRequest发送表单.

- 构造函数
`new FormData (form? : HTMLFormElement)`
- 参数
 - `form (可选)`:一个HTML表单元素,可以包含任何形式的表单控件,包括文件输入框.

- 成员方法
 - append()

    给当前FormData对象添加一个键/值对.

    ```
    void append(DOMString name, Blob value, optional DOMString filename);
    void append(DOMString name, DOMString value);

    ```
 - 参数值
    `name`: 字段名称.
    `value`: 字段值.可以是,或者一个字符串,如果全都不是,则该值会被自动转换成字符串.
    `filename`(可选): 指定文件的文件名,当value参数被指定为一个Blob对象或者一个File对象时,该文件名会被发送到服务器上,对于Blob对象来说,这个值默认为`blob`.

<!-- more -->

你可以先创建一个空的FormData对象，然后使用append()方法向该对象里添加字段，实现demo如下:

```
var oMyForm = new FormData();

oMyForm.append("username", "Groucho");
oMyForm.append("accountnum", 123456); // 数字123456被立即转换成字符串"123456"

// fileInputElement中已经包含了用户所选择的文件
oMyForm.append("userfile", fileInputElement.files[0]);

var oFileBody = '<a id="a"><b id="b">hey!</b></a>'; // Blob对象包含的文件内容
var oBlob = new Blob([oFileBody], { type: "text/xml"});

oMyForm.append("webmasterfile", oBlob);

var oReq = new XMLHttpRequest();
oReq.open("POST", "http://foo.com/submitform.php");
oReq.send(oMyForm);

```

> 注: 字段`userfile`和`webmasterfile`的值都包含了一个文件。通过 `FormData.append()`方法赋给字段`accountnum`的数字被自动转换为字符(字段的值可以是一个Blob对象，一个File对象，或者一个字符串，剩下其他类型的值都会被自动转换成字符串)。

在该例子中，我们创建了一个名为oMyForm的FormData对象，该对象中包含了名为`username`, `accountnum`, `userfile` 以及 `webmasterfile`的字段名，然后使用`XMLHttpRequest`的`send()`方法把这些数据发送了出去。`webmasterfile`字段的值不是一个字符串，还是一个Blob对象。


## HTML表单初始化FormData

可以用一个已有的`<form>`元素来初始化FormData对象,只需要把这个form元素作为参数传入FormData构造函数即可:

`var newFormData = new FormData(someFormElement);`

例如:

```
var formElement = document.getElementById("myFormElement");
var oReq = new XMLHttpRequest();
oReq.open("POST", "submitform.php");
oReq.send(new FormData(formElement));

```
你还可以在已有表单数据的基础上,继续添加新的键值对,如下:

```
var formElement = document.getElementById("myFormElement");
formData = new FormData(formElement);
formData.append("serialnumber", serialNumber++);
oReq.send(formData);

```
你可以通过这种方式添加一些不想让用户编辑的固定字段,然后再发送.

## 使用FormData发送文件

你还可以使用FormData来发送二进制文件.首先在HTML中要有一个包含了文件输入框的form元素:

```
<form enctype="multipart/form-data" method="post" name="fileinfo">
  <label>Your email address:</label>
  <input type="email" autocomplete="on" autofocus name="userid" placeholder="email" required size="32" maxlength="64" /><br />
  <label>Custom file label:</label>
  <input type="text" name="filelabel" size="12" maxlength="32" /><br />
  <label>File to stash:</label>
  <input type="file" name="file" required />
</form>
<div id="output"></div>
<a href="javascript:sendForm()">Stash the file!</a>

```
然后你就可以使用下面的代码来异步的上传用户所选择的文件:

```
function sendForm() {
  var oOutput = document.getElementById("output");
  var oData = new FormData(document.forms.namedItem("fileinfo"));

  oData.append("CustomField", "This is some extra data");

  var oReq = new XMLHttpRequest();
  oReq.open("POST", "stash.php", true);
  oReq.onload = function(oEvent) {
    if (oReq.status == 200) {
      oOutput.innerHTML = "Uploaded!";
    } else {
      oOutput.innerHTML = "Error " + oReq.status + " occurred uploading your file.<br \/>";
    }
  };

  oReq.send(oData);
}

```
你还可以不借助HTML表单,直接向FormData对象中添加一个File对象或者一个Blob对象:

```
data.append("myfile", myBlob);

```
如果FormData对象中的某个字段值是一个Blob对象，则在发送http请求时，代表该Blob对象所包含文件的文件名的`Content-Disposition`请求头的值在不同的浏览器下有所不同，Firefox使用了固定的字符串`blob`，而Chrome使用了一个随机字符串。

你还可以使用jQuery来发送FormData,但必须要正确的设置相关选项:

```
var fd = new FormData(document.getElementById("fileinfo"));
fd.append("CustomField", "This is some extra data");
$.ajax({
  url: "stash.php",
  type: "POST",
  data: fd,
  processData: false,  // 告诉jQuery不要去处理发送的数据
  contentType: false   // 告诉jQuery不要去设置Content-Type请求头
});

```
> 注：不要显式地设置`contentType`为`multipart/form-data`。


## FormData异步上传大文件

下面给出一个完整的使用`FormData`以及`XMLHttpRequest`上传大文件的例子：

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>大文件上传实例</title>
    <script type="text/javascript">
        const BYTES_PER_CHUNK = 1024 * 1024; // 每个文件切片大小定为1MB .
        var slices;
        var totalSlices;

        //发送请求
        function sendRequest() {

            var blob = document.getElementById('file').files[0];

            var start = 0;
            var end;
            var index = 0;

            // 计算文件切片总数
            slices = Math.ceil(blob.size / BYTES_PER_CHUNK);
            totalSlices= slices;

            while(start < blob.size) {
                end = start + BYTES_PER_CHUNK;
                if(end > blob.size) {
                    end = blob.size;
                }

                uploadFile(blob, index, start, end);

                start = end;
                index++;
            }
        }

        //上传文件
        function uploadFile(blob, index, start, end) {
            var xhr;
            var fd;
            var chunk;

            xhr = new XMLHttpRequest();
            xhr.onreadystatechange = function() {
                if(xhr.readyState == 4) {
                    if(xhr.responseText) {
                        alert(xhr.responseText);
                    }

                    slices--;

                    // 如果所有文件切片都成功发送，发送文件合并请求。
                    if(slices == 0) {
                        mergeFile(blob);
                        alert('文件上传完毕');
                    }
                }
            };


            chunk =blob.slice(start,end);//切割文件

            //构造form数据
            fd = new FormData();
            fd.append("file", chunk);
            fd.append("name", blob.name);
            fd.append("index", index);

            xhr.open("POST", "upload.php", true);
            xhr.send(fd);
        }

        function mergeFile(blob) {
            var xhr;
            var fd;

            xhr = new XMLHttpRequest();

            fd = new FormData();
            fd.append("name", blob.name);
            fd.append("index", totalSlices);

            xhr.open("POST", "merge.php", true);
            xhr.send(fd);
        }

    </script>
</head>
<body>

<input type="file" id="file"/>
<button  onclick="sendRequest()">上传</button>
</body>
</html>

```

- `upload.php`文件：

```
<?php
//省略了文件接收判断isset部分
//当前目录下建立一个uploads文件夹
//接收文件名时进行转码，防止中文乱码。
$target = "uploads/" .$_POST["name"] . '-' . $_POST['index'];
move_uploaded_file($_FILES['file']['tmp_name'], $target);

// Might execute too quickly.
sleep(1);

```

- `merge.php`文件：

```

<?php
//文件合并
$target = "uploads/" .iconv("utf-8","gbk",$_POST["name"]);
$dst = fopen($target, 'wb');

for($i = 0; $i < $_POST['index']; $i++) {
    $slice = $target . '-' . $i;
    $src = fopen($slice,'rb');
    stream_copy_to_stream($src, $dst);
    fclose($src);
    unlink($slice);
}

fclose($dst);

```
