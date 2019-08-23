---
title: 搭建PhoneGap的开发环境
date: 2014-05-09 09:18:35
tags:
- 应用实践
---

 
由 王宇 原创并发布 ： 
 
一 Cordova介绍
 
     Cordova是一个开源的移动开发框架。它允许你使用标准WEB技术跨平台开发，例如HTML5, CSS3 和JavaScript。回避每种移动平台自己的开发语言。在每个平台上，应用执行被封装的目标程序，通过标准API去访问每种设备的传感器、数据和网路状态。（翻译自PhoneGap的官方网站）
 
二 软件环境
 
     1、Win7-64 
     2、JDK 1.6.0_45
     3、ADT 
               Build：v22.6.2-1085508
     4、PhoneGap 2.9.1
     5、Cordova 3.4
 
三 安装 PhoneGap
 
     1、 下载安装NodeJS: 地址：http://nodejs.org/
     2、 在命令行中执行：
            C:\> npm install -g phonegap
            C:\> npm install -g cordova 
      3、配置环境变量PATH
             将以下内容添加到环境变量PATH中：
                    ;C:\Development\adt-bundle\sdk\platform-tools;C:\Development\adt-bundle\sdk\tools
                    ;%JAVA_HOME%\bin;%ANT_HOME%\bin
 
四 创建APP
             cd D:\PhonegapTest
             D:\PhonegapTest> cordova create hello com.example.hello HelloWorld
 
五 增加 Platforms
              cd D:\PhonegapTest\hello
              D:\PhonegapTest\hello> cordova platform add android
 
六 编译APP
              D:\PhonegapTest\hello> cordova build 
 
七 配置Andriod模拟器
     
     在Eclipse中， Tools->Manage AVDs   点击New Device... 配置下图内容：
 

 
点击OK
 
 
八 在模拟器中启动App HelloWorld
 
     在Eclipse中打开模拟器：
 

 
 
     在命令行中执行：
                       D:\PhonegapTest\hello> cordova emulate android
          

 
 
 
点击这个App：
 
 

 
 
九 插件(Plugin)
 
     1、使用本地通知对话框
 
          (1) 下载Dialog插件
 
                      D:\PhonegapTest\hello> cordoca plugin add org.apache.cordova.dialogs
 
 
          (2) 编辑D:\PhonegapTest\hello\www\index.html  增加一个测试按键：
 
                         <input onClick="testDialog()" type="button" value="TestButton"/>
 
          (3)使用Javascript 调用插件的API：
                         <script language='javascript'>
                                       function testDialog()
                                        {
                                                 navigator.notification.alert("This is message", null, "title", 'OK'); 
                                         }
                           </script>
          (4)运行效果


 
 
点击"TestButton"


 
 
     2、文件的写入和读取
 
          (1) 下载File插件：

                D:\PhonegapTest\hello> cordova plugin add org.apache.cordova.file
 
          (2) 配置插件：
               
                 在config.xml中增加下列代码：
 
 <preference name="AndroidExtraFilesystems" value="files,files-external,documents,sdcard,cache,cache-external,root" />
 
          (3) 编辑文件index_file.html
<html>
    <body>
        <form onsubmit="return saveText()">
            <label for="name">Name</label><br>
            <input id="name" /><br>
 
            <label for="desc">Description</label><br>
            <input id="desc" /><br>
 
            <input type="submit" value="Save" />
        </form>
 
 
        <dl id="definitions">
 
        </dl>
    </body>
    <script type="text/javascript" src="cordova.js"></script>
    <script>
        var FILENAME = 'database.db',
            $ = function (id) {
                return document.getElementById(id);
            },
            failCB = function (msg) {
                return function () {
                    alert('[FAIL] ' + msg);
                }
            },
            file = {
                writer: { available: false },
                reader: { available: false }
            },
            dbEntries = [];
 
        document.addEventListener('deviceready', function () {
            var fail = failCB('requestFileSystem');
            window.requestFileSystem(LocalFileSystem.PERSISTENT, 0, gotFS, fail);
        }, false);
 
        function gotFS(fs) {
            var fail = failCB('getFile');
            fs.root.getFile(FILENAME, {create: true, exclusive: false},
                            gotFileEntry, fail);
        }
 
        function gotFileEntry(fileEntry) {
            var fail = failCB('createWriter');
            file.entry = fileEntry;
 
            fileEntry.createWriter(gotFileWriter, fail);
            readText();
        }
 
        function gotFileWriter(fileWriter) {
            file.writer.available = true;
            file.writer.object = fileWriter;
        }
 
        function saveText(e) {
            var name = $('name').value,
                desc = $('desc').value,
                fail;
 
            dbEntries.push('<dt>' + name + '</dt><dd>' + desc + '</dd>')
            $('name').value = '';
            $('desc').value = '';
 
            $('definitions').innerHTML = dbEntries.join('');
 
            if (file.writer.available) {
                file.writer.available = false;
                file.writer.object.onwriteend = function (evt) {
                    file.writer.available = true;
                    file.writer.object.seek(0);
                }
                file.writer.object.write(dbEntries.join("\n"));
            }
 
            return false;
        }
 
        function readText() {
            if (file.entry) {
                file.entry.file(function (dbFile) {
                    var reader = new FileReader();
                    reader.onloadend = function (evt) {
                        var textArray = evt.target.result.split("\n");
 
                        dbEntries = textArray.concat(dbEntries);
 
                        $('definitions').innerHTML = dbEntries.join('');
                    }
                    reader.readAsText(dbFile);
                }, failCB("FileReader"));
            }
 
            return false;
        }
    </script>
</html>
 
      (4) 修改程序启动的首界面：
 
          将config.xml中的<content src="index.html" /> 修改为：<content src="index_file.html" />
 
    (5) 运行效果


 

        在文本框中输入数据。被保存的数据将会被重新读取，并显示在Save按键下面。

        3、多媒体插件（播放mp3）
 
          (1) 下载Media插件：
                    D:\PhonegapTest\hello> cordova plugin add org.apache.cordova.media

          (2) Javascript API:
 
 * media.getCurrentPosition: Returns the current position within an audio file.(返回当前音频文件的路径)
 * media.getDuration: Returns the duration of an audio file.（返回音频文件播放的持续时间）
 * media.play: Start or resume playing an audio file.(停止或恢复一个音频文件)
 * media.pause: Pause playback of an audio file.(暂停一个音频文件)
 * media.release: Releases the underlying operating system's audio resources.(释放操作系统的底层资源，即释放内存。)
 * media.seekTo: Moves the position within the audio file.(移动到音频文件的指定播放位置)
 * media.setVolume: Set the volume for audio playback.(设置音频文件的音量)
 * media.startRecord: Start recording an audio file.(开始录制一个音频文件)
 * media.stopRecord: Stop recording an audio file.(停止录制一个音频文件)
 * 
media.stop: Stop playing an audio file.(停止播发一个音频文件)
 
                  (3)将一个test.mp3 放置到下面这路径中：
                              D:\PhonegapTest\hello\www

                  (4)编译文件audio.html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
                      "http://www.w3.org/TR/html4/strict.dtd">
<html>
  <head>
    <title>PhoneGap Back Button Example</title>
 
    <script type="text/javascript" charset="utf-8" src="cordova.js"></script>
    <script type="text/javascript" charset="utf-8">
   
    var myMedia = null;
	var playing = false;
	
    function playAudio() {
		if (!playing) {
                        myMedia.play();	
			document.getElementById('play').src = "img/pause.png";
			playing = true;	
		} else {
			myMedia.pause();
                        document.getElementById('play').src = "img/play.png";    
                        playing = false; 
		}
    }
 
    function stopAudio() {
		myMedia.stop();
		playing = false;
                document.getElementById('play').src = "img/play.png";    
		document.getElementById('audio_position').innerHTML = "0.000 sec";
	}
 
   function onLoad() {
       document.addEventListener("deviceready", onDeviceReady, false);
   }
   
   function onDeviceReady(){
   	console.log("Got device ready");
   	updateMedia();
   }
   
   function updateMedia(src) {
   	    // Clean up old file
   	    if (myMedia != null) {
			myMedia.release();
		}
		
		// Get the new media file
		var yourSelect = document.getElementById('playlist');		
                myMedia = new Media(yourSelect.options[yourSelect.selectedIndex].value, stopAudio, null);
 
		// Update media position every second
	        var mediaTimer = setInterval(function() {
	        // get media position
	        myMedia.getCurrentPosition(
	            // success callback
	            function(position) {
	                if (position > -1) {
						document.getElementById('audio_position').innerHTML = (position/1000) + " sec";
	                }
	            },
	            // error callback
	            function(e) {
	                console.log("Error getting pos=" + e);
	            }
	        );
	    }, 1000);
   }
 
   function setAudioPosition(position) {
       document.getElementById('audio_position').innerHTML =position;
   }
 </script>
 <body onload="onLoad()">
   <h1>Audio Player</h1>
   <p id="audio_position">0.000 sec</p>
   <p>
	   <select id="playlist" onchange="updateMedia()">
	        <option checked value="/android_asset/www/test.mp3">Asset</option>
	        <option value="test.mp3">SD Card</option>
	        <option value="http://audio.ibeat.org/content/p1rj1s/p1rj1s_-_rockGuitar.mp3">Internet</option>
	   </select>
   </p>
   <a href="#" onclick="playAudio()"><img id="play" src="img/play.png"></a>
   <a href="#" onclick="stopAudio()"><img id="stop" src="img/stop.png"></a>
 </body>
</html>


 

       (5) 修改程序启动的首界面：
          将config.xml中的<content src="index.html" /> 修改为：<content src="audio.html" />
 
       (6) 运行效果


 


 
 
Asset      是播放www路径下的test.mp3
SD Card  是播放SD Card 根目录下的test.mp3
Internet   是播放URL指定的mp3(http://audio.ibeat.org/content/p1rj1s/p1rj1s_-_rockGuitar.mp3)
 

     4、通知插件(Notification)
 
          (1) 安装通知插件
               首先安装Git，Windows版本安装包的下载地址：http://msysgit.github.com/
               将Git的命令路径(C:\Program Files (x86)\Git\bin)添加到环境变量PATH中
               
               执行下列命令安装Notification plugin:
                   D:\PhonegapTest\hello>  cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-vibration.git
                   D:\PhonegapTest\hello>  cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-dialogs.git
                   D:\PhonegapTest\hello>  cordova plugin rm org.apache.cordova.core.dialogs
                   D:\PhonegapTest\hello>  cordova plugin rm org.apache.cordova.core.vibration
               
         (2) Javascript API:
               Methods
                    notification.alert
                    notification.confirm
                    notification.prompt
                    notification.beep
                    notification.vibrate
 
          (3) 编辑文件notification.html
                   
<!DOCTYPE html>
<html>
  <head>
    <title>Notification Example</title>

    <script type="text/javascript" charset="utf-8" src="cordova.js"></script>
    <script type="text/javascript" charset="utf-8">

    // Wait for device API libraries to load
    //
    document.addEventListener("deviceready", onDeviceReady, false);

    // device APIs are available
    //
    function onDeviceReady() {
        // Empty
    }

    // Show a custom alert
    //
    function showAlert() {
        navigator.notification.alert(
            'You are the winner!',  // message
            'Game Over',            // title
            'Done'                  // buttonName
        );
    }

    // Beep three times
    //
    function playBeep() {
        navigator.notification.beep(3);
    }

    // Vibrate for 2 seconds
    //
    function vibrate() {
        navigator.notification.vibrate(2000);
    }

    </script>
  </head>
  <body>
    <p><a href="#" onclick="showAlert(); return false;">Show Alert</a></p>
    <p><a href="#" onclick="playBeep(); return false;">Play Beep</a></p>
    <p><a href="#" onclick="vibrate(); return false;">Vibrate</a></p>
  </body>
</html>
 
          (5) 修改程序启动的首界面：
               将config.xml中的<content src="index.html" /> 修改为：<content src="notification.html" />
 
          (6) 运行效果
 

 


 
 

 十 开发工具
 
     (1) 设计工具

          UI设计采用Adobe Photoshop CS6, 详见下图


 


 
    (2) 代码工具
 
          PhoneGap的代码环境是比较繁杂的，所以没有一个单一的工具可以完全涵盖它的代码编辑工作。传统的方式可以采用VIM EMACS等。在PhoneGap中大部分的工作都集中在HTML Javascrip 和CSS上，这里推荐一个IDE的编辑工具：WebStorm 。 对于Plugin， Android中采用 ADT(Eclipse), iOS 中采用Xcode.
 
          WebStorm界面如下：


 
 

十一 调试
 
     关于PhoneGap的调试，先说一个失败的方案，这个方案折磨的我好苦，即在Chrome浏览器中安装插件Ripple Emulator(Beta) 0.9.15(官网上的版本)，它的主要问题是，只支持Cordova2.0， 但是目前的Cordova版本是3.4  。 当调试的时候，Chrome处于死锁等待状态。 后来在stack overflow的一个回复中，找到了一个可行的方法。在阐述这个方法之前，先说一下PhoneGap Emulator, 即它允许你在桌面浏览器中运行和调试PhoneGap的应用程序。
 
     (1) 安装Ripple
          
               D:\PhonegapTest\hello> npm install -g ripple-emulator
 
     (2) 运行
 
               D:\PhonegapTest\hello> ripple emulate
 
               执行以上命令后，如果Chrome是默认浏览器，将会自动在浏览器中显示。URL：http://localhost:4400/?enableripple=cordova-3.0.0
 
 

 
     (3) 打开Chrome Developer Tools


 
     (4)在47行处设置断点，并点击"TestButton"
 

 


 






