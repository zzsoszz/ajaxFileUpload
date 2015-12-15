# ajaxFileUpload

http://blog.csdn.net/it_man/article/details/43800957
jquery  ajaxFileUpload
 jquery插件--ajaxfileupload.js上传文件原理分析
标签： jquery插件fileupload
2015-02-13 21:18 8985人阅读 评论(0) 收藏 举报
分类：
javaScript（18）

版权声明：本文为博主原创文章，未经博主允许不得转载。
英文注解应该是原作者写的吧~说实话，有些if判断里的东西我也没太弄明白，但是大致思路还是OK的。
[javascript] view plaincopy在CODE上查看代码片派生到我的代码片

    jQuery.extend({  
        createUploadIframe: function (id, uri) {//id为当前系统时间字符串，uri是外部传入的json对象的一个参数  
            //create frame  
            var frameId = 'jUploadFrame' + id; //给iframe添加一个独一无二的id  
            var iframeHtml = '<iframe id="' + frameId + '" name="' + frameId + '" style="position:absolute; top:-9999px; left:-9999px"'; //创建iframe元素  
            if (window.ActiveXObject) {//判断浏览器是否支持ActiveX控件  
                if (typeof uri == 'boolean') {  
                    iframeHtml += ' src="' + 'javascript:false' + '"';  
                }            else if (typeof uri == 'string') {  
                    iframeHtml += ' src="' + uri + '"';  
                }  
            }  
            iframeHtml += ' />';  
            jQuery(iframeHtml).appendTo(document.body); //将动态iframe追加到body中  
            return jQuery('#' + frameId).get(0); //返回iframe对象  
        },  
        createUploadForm: function (id, fileElementId, data) {//id为当前系统时间字符串，fileElementId为页面<input type='file' />的id，data的值需要根据传入json的键来决定  
            //create form      
            var formId = 'jUploadForm' + id; //给form添加一个独一无二的id  
            var fileId = 'jUploadFile' + id; //给<input type='file' />添加一个独一无二的id  
            var form = jQuery('<form  action="" method="POST" name="' + formId + '" id="' + formId + '" enctype="multipart/form-data" ></form>'); //创建form元素  
            if (data) {//通常为false  
                for (var i in data) {  
                    jQuery('<input type="hidden" name="' + i + '" value="' + data[i] + '" />').appendTo(form); //根据data的内容，创建隐藏域，这部分我还不知道是什么时候用到。估计是传入json的时候，如果默认传一些参数的话要用到。  
                }  
            }        var oldElement = jQuery('#' + fileElementId); //得到页面中的<input type='file' />对象  
            var newElement = jQuery(oldElement).clone(); //克隆页面中的<input type='file' />对象  
            jQuery(oldElement).attr('id', fileId); //修改原对象的id  
            jQuery(oldElement).before(newElement); //在原对象前插入克隆对象  
            jQuery(oldElement).appendTo(form); //把原对象插入到动态form的结尾处  
            //set attributes  
            jQuery(form).css('position', 'absolute'); //给动态form添加样式，使其浮动起来，  
            jQuery(form).css('top', '-1200px');  
            jQuery(form).css('left', '-1200px');  
            jQuery(form).appendTo('body'); //把动态form插入到body中  
            return form;  
        },  
        ajaxFileUpload: function (s) {//这里s是个json对象，传入一些ajax的参数  
            // TODO introduce global settings, allowing the client to modify them for all requests, not only timeout          
            s = jQuery.extend({}, jQuery.ajaxSettings, s); //此时的s对象是由jQuery.ajaxSettings和原s对象扩展后的对象  
            var id = new Date().getTime(); //取当前系统时间，目的是得到一个独一无二的数字  
            var form = jQuery.createUploadForm(id, s.fileElementId, (typeof (s.data) == 'undefined' ? false : s.data)); //创建动态form  
            var io = jQuery.createUploadIframe(id, s.secureuri); //创建动态iframe  
            var frameId = 'jUploadFrame' + id; //动态iframe的id  
            var formId = 'jUploadForm' + id; //动态form的id  
            // Watch for a new set of requests  
            if (s.global && !jQuery.active++) {//当jQuery开始一个ajax请求时发生  
                jQuery.event.trigger("ajaxStart"); //触发ajaxStart方法  
            }        var requestDone = false; //请求完成标志  
            // Create the request object  
            var xml = {};        if (s.global)  
                jQuery.event.trigger("ajaxSend", [xml, s]); //触发ajaxSend方法  
            // Wait for a response to come back  
            var uploadCallback = function (isTimeout) {//回调函数  
                var io = document.getElementById(frameId); //得到iframe对象  
                try {                if (io.contentWindow) {//动态iframe所在窗口对象是否存在  
                        xml.responseText = io.contentWindow.document.body ? io.contentWindow.document.body.innerHTML : null;  
                        xml.responseXML = io.contentWindow.document.XMLDocument ? io.contentWindow.document.XMLDocument : io.contentWindow.document;  
                    } else if (io.contentDocument) {//动态iframe的文档对象是否存在  
                        xml.responseText = io.contentDocument.document.body ? io.contentDocument.document.body.innerHTML : null;  
                        xml.responseXML = io.contentDocument.document.XMLDocument ? io.contentDocument.document.XMLDocument : io.contentDocument.document;  
                    }  
                } catch (e) {  
                    jQuery.handleError(s, xml, null, e);  
                }            if (xml || isTimeout == "timeout") {//xml变量被赋值或者isTimeout == "timeout"都表示请求发出，并且有响应  
                    requestDone = true; //请求完成  
                    var status;                try {  
                        status = isTimeout != "timeout" ? "success" : "error"; //如果不是“超时”，表示请求成功  
                        // Make sure that the request was successful or notmodified  
                        if (status != "error") {                        // process the data (runs the xml through httpData regardless of callback)  
                            var data = jQuery.uploadHttpData(xml, s.dataType); //根据传送的type类型，返回json对象，此时返回的data就是后台操作后的返回结果  
                            // If a local callback was specified, fire it and pass it the data  
                            if (s.success)  
                                s.success(data, status); //执行上传成功的操作  
                            // Fire the global callback  
                            if (s.global)  
                                jQuery.event.trigger("ajaxSuccess", [xml, s]);  
                        } else  
                            jQuery.handleError(s, xml, status);  
                    } catch (e) {  
                        status = "error";  
                        jQuery.handleError(s, xml, status, e);  
                    }                // The request was completed  
                    if (s.global)  
                        jQuery.event.trigger("ajaxComplete", [xml, s]);                // Handle the global AJAX counter  
                    if (s.global && ! --jQuery.active)  
                        jQuery.event.trigger("ajaxStop");                // Process result  
                    if (s.complete)  
                        s.complete(xml, status);  
                    jQuery(io).unbind();//移除iframe的事件处理程序  
                    setTimeout(function () {//设置超时时间  
                        try {  
                            jQuery(io).remove();//移除动态iframe  
                            jQuery(form).remove();//移除动态form  
                        } catch (e) {  
                            jQuery.handleError(s, xml, null, e);  
                        }  
                    }, 100)  
                    xml = null  
                }  
            }        // Timeout checker  
            if (s.timeout > 0) {//超时检测  
                setTimeout(function () {                // Check to see if the request is still happening  
                    if (!requestDone) uploadCallback("timeout");//如果请求仍未完成，就发送超时信号  
                }, s.timeout);  
            }        try {            var form = jQuery('#' + formId);  
                jQuery(form).attr('action', s.url);//传入的ajax页面导向url  
                jQuery(form).attr('method', 'POST');//设置提交表单方式  
                jQuery(form).attr('target', frameId);//返回的目标iframe，就是创建的动态iframe  
                if (form.encoding) {//选择编码方式  
                    jQuery(form).attr('encoding', 'multipart/form-data');  
                }            else {  
                    jQuery(form).attr('enctype', 'multipart/form-data');  
                }  
                jQuery(form).submit();//提交form表单  
            } catch (e) {  
                jQuery.handleError(s, xml, null, e);  
            }  
            jQuery('#' + frameId).load(uploadCallback); //ajax 请求从服务器加载数据，同时传入回调函数  
            return { abort: function () { } };  
        },  
        uploadHttpData: function (r, type) {        var data = !type;  
            data = type == "xml" || data ? r.responseXML : r.responseText;        // If the type is "script", eval it in global context  
            if (type == "script")  
                jQuery.globalEval(data);        // Get the JavaScript object, if JSON is used.  
            if (type == "json")  
                eval("data = " + data);        // evaluate scripts within html  
            if (type == "html")  
                jQuery("<div>").html(data).evalScripts();        return data;  
        }  
    })  
       


ajaxfileupload.js插件大致的思路就是如上所述，但是对于ajax来说，传值也是相当关键的部分，也就是传入的json对象里的键值对。调用方法如下：
[javascript] view plaincopy在CODE上查看代码片派生到我的代码片

    $.ajaxFileUpload  
    (  
        {  
            url: '../../XXXX/XXXX.aspx', //用于文件上传的服务器端请求地址  
            secureuri: false,           //一般设置为false  
            fileElementId: $("input#xxx").attr("id"), //文件上传控件的id属性  <input type="file" id="file" name="file" /> 注意，这里一定要有name值     
                                                    //$("form").serialize(),表单序列化。指把所有元素的ID，NAME 等全部发过去  
            dataType: 'json',//返回值类型 一般设置为json  
            complete: function () {//只要完成即执行，最后执行  
            },  
            success: function (data, status)  //服务器成功响应处理函数  
            {            if (typeof (data.error) != 'undefined') {                if (data.error != '') {                    if (data.error == "1001") {//这个error（错误码）是由自己定义的，根据后台返回的json对象的键值而判断  
                        }                    else if (data.error == "1002") {  
                        }  
                        alert(data.msg);//同error  
                        return;  
                    } else {  
                        alert(data.msg);  
                    }  
                }            /*                *    这里就是做一些其他操作，比如把图片显示到某控件中去之类的。                */  
            },  
            error: function (data, status, e)//服务器响应失败处理函数  
            {  
                alert(e);  
            }  
        }  
    )  


 

整个就是使用ajaxfileupload.js插件的大致方法。当然，明白其工作原理越透彻，我们也就能越好的去操作和使用它。

以上的分析希望对刚接触ajaxfileupload.js插件的朋友们有帮助。

文件下载地址：http://files.cnblogs.com/zhouhongyu1989/ajaxfileupload.js