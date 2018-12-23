I figure we all have that team. The folks that have their special way of doing things that nobody can change because their management structure goes just high enough to thwart all efforts. Unfortunately, for me, this team was Laptop Deployment. We started standing up the CMDB, and got all of our End User devices loaded into ServiceNow, but the Deployment team didn’t like the idea of logging in to ServiceNow every time they needed to assign a laptop to somebody, and we couldn’t force them. Besides, they already had a super-snazzy Excel file.

Go ahead, roll your eyes, I know.

To be fair, this Excel file was pretty snazzy. It had two worksheets. On the first, a form would pop up and the Tech would scan a personalized barcode,

![TechEntry Form](https://github.com/Greg-Malski/Asset-Checkin-Interface/raw/master/Screenshots/UserEntry.png)

then the laptop’s Asset Tag, and a User Barcode, or some canned Action/Assignment codes, then press submit.

![AssetAction Form](https://github.com/Greg-Malski/Asset-Checkin-Interface/raw/master/Screenshots/AssetAction.png)

Some VBA script would run in the background and log everything to the second worksheet. They stored it on a shared drive so their analysts had access to it, but the whole process needed to move into ServiceNow.

Obviously, ServiceNow offers better reporting, better tracking, and one certain thing they wanted but could never attain with Excel: the ability to open the spreadsheet in more than one location. As a consequence of that, each building had their own files that had to be reconciled before comprehensive reporting could occur. This should have been an easy sell, and it would have been. They were completely on board with it, except that they didn’t want to give up their Excel file interface, and they didn’t want to have to log in to ServiceNow all the time.

Thankfully, REST APIs are a thing.

Most of you will know that ServiceNow has very robust support for REST APIs, but many people do not realize that Excel also has some support for REST APIs. So, here’s the plan:

![ThePlan: Excel to ServiceNow](https://github.com/Greg-Malski/Asset-Checkin-Interface/raw/master/Screenshots/ThePlan.png)

Our data will start in the Excel file. We’ll package it up and shoot it to ServiceNow via the REST API. Once it hits the Endpoint, we send it over to a script include to verify the data is good. If everything is good, it sends a standard 200 response back to Excel and then drops the data into a Catalog Item with a special workflow. If something is wrong, it sends a custom error status back to Excel which then prompts the user to fix the information.

If you don’t care about looking under the covers, here’s a great place to leave it. You’ve learned that it’s possible to link up Excel with ServiceNow and make actual updates to actual tables. If you want to peek under the covers with me, though, let’s continue on and walk through the pieces, together!

## The Excel File

This isn’t going to be a detailed VBA tutorial. There are already plenty of those in existance, but I do want to show off a few bits of code that make this thing a little more secure and a little more cool.

First off, let’s talk about the code required to POST and GET. I wish I could tell you exactly from where I stole this, but suffice it to say, it’s pretty available though a simple Google search.

First, you need to define a couple of variables:  
<script src="https://pastebin.com/embed_js/s7SpqvBh"><script></p> <p>Then, we need to create our GET or POST object: <script src="https://pastebin.com/embed_js/93C2HzmA"></script> <style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.vbscript .de1, .vbscript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.vbscript {color:#ACACAC;}.vbscript .imp {font-weight: bold; color: red;}.vbscript li, .vbscript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.vbscript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.vbscript .kw1 {color: #F660AB; font-weight: bold;}.vbscript .kw2 {color: #E56717; font-weight: bold;}.vbscript .kw3 {color: #8D38C9; font-weight: bold;}.vbscript .kw4 {color: #151B8D; font-weight: bold;}.vbscript .co1 {color: #008000;}.vbscript .es0 {color: #800000; font-weight: bold;}.vbscript .st0 {color: #800000;}.vbscript .ln-xtra, .vbscript li.ln-xtra, .vbscript div.ln-xtra {background:#FFFF88;}.vbscript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/s7SpqvBh) - [See Original](https://pastebin.com/s7SpqvBh)</div>

1.  <div class="de1"><span class="kw3">Dim</span> objHTTP As Object</div>

2.  <div class="de1"><span class="kw3">Dim</span> Json As <span class="kw2">String</span></div>

3.  <div class="de1"><span class="kw3">Dim</span> result As <span class="kw2">String</span></div>

</div>

In our instance, this is all rolled into a Scoped App, so you'll end up replacing *company* with your own company. Also, you'll see the variable "env". This is normally where your subdomain for ServiceNow would be. Since I have versions of this for our Production as well as subproduction environments, I have a variable in place to make swapping between them easy. The API URL in this example GETs a property variable from ServiceNow to be sure the user is using the right version of the Interface. It's fairly basic version control, but it's been effective so far.

Next, you'll set your Headers. We're using Basic Authentication. I've replaced our actual token with the lame word TOKEN:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.vbscript .de1, .vbscript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.vbscript {color:#ACACAC;}.vbscript .imp {font-weight: bold; color: red;}.vbscript li, .vbscript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.vbscript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.vbscript .kw1 {color: #F660AB; font-weight: bold;}.vbscript .kw2 {color: #E56717; font-weight: bold;}.vbscript .kw3 {color: #8D38C9; font-weight: bold;}.vbscript .kw4 {color: #151B8D; font-weight: bold;}.vbscript .co1 {color: #008000;}.vbscript .es0 {color: #800000; font-weight: bold;}.vbscript .st0 {color: #800000;}.vbscript .ln-xtra, .vbscript li.ln-xtra, .vbscript div.ln-xtra {background:#FFFF88;}.vbscript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/FP7bV3ev) - [See Original](https://pastebin.com/FP7bV3ev)</div>

1.  <div class="de1">objHTTP.SetRequestHeader <span class="st0">"Content-Type"</span>, <span class="st0">"application/json"</span></div>

2.  <div class="de1">objHTTP.SetRequestHeader <span class="st0">"Authorization"</span>, <span class="st0">"Basic TOKEN"</span></div>

</div>

Finally, you'll send it off and receive the response:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.vbscript .de1, .vbscript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.vbscript {color:#ACACAC;}.vbscript .imp {font-weight: bold; color: red;}.vbscript li, .vbscript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.vbscript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.vbscript .kw1 {color: #F660AB; font-weight: bold;}.vbscript .kw2 {color: #E56717; font-weight: bold;}.vbscript .kw3 {color: #8D38C9; font-weight: bold;}.vbscript .kw4 {color: #151B8D; font-weight: bold;}.vbscript .co1 {color: #008000;}.vbscript .es0 {color: #800000; font-weight: bold;}.vbscript .st0 {color: #800000;}.vbscript .ln-xtra, .vbscript li.ln-xtra, .vbscript div.ln-xtra {background:#FFFF88;}.vbscript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/NXFmFFFX) - [See Original](https://pastebin.com/NXFmFFFX)</div>

1.  <div class="de1">objHTTP.Send (Json)</div>

2.  <div class="de1">StatusNum <span class="sy0">=</span> objHTTP.Status</div>

3.  <div class="de1">Status <span class="sy0">=</span> objHTTP.StatusText</div>

4.  <div class="de1">result <span class="sy0">=</span> objHTTP.ResponseText</div>

</div>

Our Excel Interface uses this scripting a few times:

1.  On open, the interface checks it's version number against a property stored within ServiceNow
2.  The Technician's barcode is compared against an Authorization Table within the Scoped App and their name is returned
3.  The asset movement data is sent to ServiceNow and a completion or failure code is returned

Another cool feature of the Excel interface involves receiving the Response. Excel doesn't have a built-in JSON parser, but it is able to separate out the Status Codes.  
For the full Excel Interface, check it out here on GitHub. To access the VBA console, open the Excel File, ignore the error message caused by a bad URL, and hit Alt+F11.  
A few other features of the Excel Spreadsheet:

*   Local Logging
*   Stockroom Support

## The Scripted REST Endpoint

Once the Excel file packages up the update and shoots it over to ServiceNow, the package is caught by a Scripted REST Endpoint. This Endpoint is pretty simple and consists of two parts:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.javascript .de1, .javascript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.javascript {color:#ACACAC;}.javascript .imp {font-weight: bold; color: red;}.javascript li, .javascript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.javascript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.javascript .kw1 {color: #000066; font-weight: bold;}.javascript .kw2 {color: #003366; font-weight: bold;}.javascript .kw3 {color: #000066;}.javascript .kw5 {color: #FF0000;}.javascript .co1 {color: #006600; font-style: italic;}.javascript .co2 {color: #009966; font-style: italic;}.javascript .coMULTI {color: #006600; font-style: italic;}.javascript .es0 {color: #000099; font-weight: bold;}.javascript .br0 {color: #009900;}.javascript .sy0 {color: #339933;}.javascript .st0 {color: #3366CC;}.javascript .nu0 {color: #CC0000;}.javascript .me1 {color: #660066;}.javascript .ln-xtra, .javascript li.ln-xtra, .javascript div.ln-xtra {background:#FFFF88;}.javascript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/WZWDxpXx) - [See Original](https://pastebin.com/WZWDxpXx)</div>

1.  <div class="de1"><span class="br0">(</span><span class="kw1">function</span> process<span class="br0">(</span><span class="coMULTI">/*RESTAPIRequest*/</span> request<span class="sy0">,</span> <span class="coMULTI">/*RESTAPIResponse*/</span> response<span class="br0">)</span> <span class="br0">{</span></div>

2.  <div class="de1">    <span class="co1">//Receive Asset Data</span></div>

3.  <div class="de1">    <span class="kw1">var</span> j <span class="sy0">=</span> request.<span class="me1">body</span>.<span class="me1">data</span><span class="sy0">;</span></div>

5.  <div class="de2">    gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Asset Movement Received'</span> <span class="sy0">,</span> <span class="st0">'SSB'</span><span class="br0">)</span><span class="sy0">;</span></div>

6.  <div class="de1">    gs.<span class="me1">debug</span><span class="br0">(</span><span class="st0">'Asset Movement Body Data: '</span> <span class="sy0">+</span> JSON.<span class="me1">stringify</span><span class="br0">(</span>j<span class="br0">)</span> <span class="sy0">,</span> <span class="st0">'SSB'</span><span class="br0">)</span><span class="sy0">;</span></div>

8.  <div class="de1">    <span class="kw1">var</span> foo <span class="sy0">=</span> <span class="kw1">new</span> x_ihgih_ssb_api.<span class="me1">assetVeritas</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

10.  <div class="de2">    <span class="co1">//Send Response</span></div>

11.  <div class="de1">    <span class="kw1">var</span> statusReturned <span class="sy0">=</span> foo.<span class="me1">veritasReply</span><span class="br0">(</span>j<span class="br0">)</span><span class="sy0">;</span></div>

13.  <div class="de1">    gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Completing Processing'</span> <span class="sy0">,</span> <span class="st0">'SSB'</span><span class="br0">)</span><span class="sy0">;</span></div>

14.  <div class="de1">    gs.<span class="me1">debug</span><span class="br0">(</span><span class="st0">'REST Endpoint completed. Returned: '</span> <span class="sy0">+</span> JSON.<span class="me1">stringify</span><span class="br0">(</span>statusReturned<span class="br0">)</span> <span class="sy0">,</span> <span class="st0">'SSB'</span><span class="br0">)</span><span class="sy0">;</span></div>

16.  <div class="de1">    response.<span class="me1">setContentType</span><span class="br0">(</span><span class="st0">'application/json'</span><span class="br0">)</span><span class="sy0">;</span></div>

17.  <div class="de1">    response.<span class="me1">setStatus</span><span class="br0">(</span>statusReturned.<span class="me1">http_status</span><span class="br0">)</span><span class="sy0">;</span></div>

19.  <div class="de1">    <span class="kw1">var</span> writer <span class="sy0">=</span> response.<span class="me1">getStreamWriter</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

20.  <div class="de2">    writer.<span class="me1">writeString</span><span class="br0">(</span>JSON.<span class="me1">stringify</span><span class="br0">(</span>statusReturned<span class="br0">)</span><span class="br0">)</span><span class="sy0">;</span></div>

22.  <div class="de1">    <span class="br0">}</span><span class="br0">)</span><span class="br0">(</span>request<span class="sy0">,</span> response<span class="br0">)</span><span class="sy0">;</span></div>

</div>

Part 1 is where the Asset Data is received. We set the payload to variable "j", then pass it on to a script include for verification.

Once the script include has completed the verification, it either passes the data off to the next step (and returns a standard Status 200), or kicks back one of a few custom error codes. Then, we move on to Part 2, which composes the response and sends it back to Excel. When Excel receives the response, it uses a simple if chain to decide to what to do next. It will either move on, or freeze and wait for the user to fix the data:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.vbscript .de1, .vbscript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.vbscript {color:#ACACAC;}.vbscript .imp {font-weight: bold; color: red;}.vbscript li, .vbscript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.vbscript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.vbscript .kw1 {color: #F660AB; font-weight: bold;}.vbscript .kw2 {color: #E56717; font-weight: bold;}.vbscript .kw3 {color: #8D38C9; font-weight: bold;}.vbscript .kw4 {color: #151B8D; font-weight: bold;}.vbscript .co1 {color: #008000;}.vbscript .es0 {color: #800000; font-weight: bold;}.vbscript .st0 {color: #800000;}.vbscript .ln-xtra, .vbscript li.ln-xtra, .vbscript div.ln-xtra {background:#FFFF88;}.vbscript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/CxkwN9kf) - [See Original](https://pastebin.com/CxkwN9kf)</div>

1.  <div class="de1"><span class="co1">'Print Response</span></div>

2.  <div class="de1"><span class="kw3">Dim</span> responseError As <span class="kw2">String</span></div>

4.  <div class="de1"><span class="kw3">If</span> StatusNum <span class="sy0">=</span> <span class="st0">"200"</span> <span class="kw3">Then</span></div>

5.  <div class="de2">    DataEntry.Range(<span class="st0">"G25"</span>).Value <span class="sy0">=</span> <span class="st0">"Asset update successful!"</span></div>

6.  <div class="de1">    responseError <span class="sy0">=</span> <span class="kw1">False</span></div>

7.  <div class="de1">ElseIf StatusNum <span class="sy0">=</span> <span class="st0">"429"</span> <span class="kw3">Then</span></div>

8.  <div class="de1">    <span class="kw2">MsgBox</span> <span class="st0">"ServiceNow is currently experiencing an overload. Please try again in a moment."</span> <span class="sy0">&</span> <span class="kw1">vbNewLine</span> <span class="sy0">&</span> <span class="st0">"If this problem persists, use the Failover Request"</span></div>

9.  <div class="de1">    responseError <span class="sy0">=</span> <span class="kw1">True</span></div>

10.  <div class="de2">ElseIf StatusNum <span class="sy0">=</span> <span class="st0">"460"</span> <span class="kw3">Then</span></div>

11.  <div class="de1">    <span class="kw2">MsgBox</span> <span class="st0">"The Serial Number received did not match any assets. Please try again."</span></div>

12.  <div class="de1">    responseError <span class="sy0">=</span> <span class="kw1">True</span></div>

13.  <div class="de1">    Status <span class="sy0">=</span> <span class="st0">"Invalid Serial Number"</span></div>

14.  <div class="de1">ElseIf StatusNum <span class="sy0">=</span> <span class="st0">"461"</span> <span class="kw3">Then</span></div>

15.  <div class="de2">    <span class="kw2">MsgBox</span> <span class="st0">"The Action code received was invalid. Please try again."</span></div>

16.  <div class="de1">    responseError <span class="sy0">=</span> <span class="kw1">True</span></div>

17.  <div class="de1">    Status <span class="sy0">=</span> <span class="st0">"Invalid Action Code"</span></div>

18.  <div class="de1">ElseIf StatusNum <span class="sy0">=</span> <span class="st0">"462"</span> <span class="kw3">Then</span></div>

19.  <div class="de1">    <span class="kw2">MsgBox</span> <span class="st0">"The Assignment code received was invalid or the Correlation ID received did not match any users. Please try again."</span></div>

20.  <div class="de2">    responseError <span class="sy0">=</span> <span class="kw1">True</span></div>

21.  <div class="de1">    Status <span class="sy0">=</span> <span class="st0">"Invalid Assignment or User"</span></div>

22.  <div class="de1">ElseIf StatusNum <span class="sy0">=</span> <span class="st0">"500"</span> <span class="kw3">Then</span></div>

23.  <div class="de1">    <span class="kw2">MsgBox</span> <span class="st0">"ServiceNow experienced an error. Please contact support if this persists."</span></div>

24.  <div class="de1">    responseError <span class="sy0">=</span> <span class="kw1">True</span></div>

25.  <div class="de2">    Status <span class="sy0">=</span> <span class="st0">"ServiceNow Error"</span></div>

26.  <div class="de1"><span class="kw3">Else</span></div>

27.  <div class="de1">    DataEntry.Range(<span class="st0">"G25"</span>).Value <span class="sy0">=</span> Status <span class="sy0">&</span> <span class="st0">": "</span> <span class="sy0">&</span> result</div>

28.  <div class="de1"><span class="kw3">End</span> <span class="kw3">If</span></div>

</div>

## The Script Include

The Script Include is where all the real work happens, at least on the verification side. For simplicity, we're going to stick with calling the payload j:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.javascript .de1, .javascript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.javascript {color:#ACACAC;}.javascript .imp {font-weight: bold; color: red;}.javascript li, .javascript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.javascript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.javascript .kw1 {color: #000066; font-weight: bold;}.javascript .kw2 {color: #003366; font-weight: bold;}.javascript .kw3 {color: #000066;}.javascript .kw5 {color: #FF0000;}.javascript .co1 {color: #006600; font-style: italic;}.javascript .co2 {color: #009966; font-style: italic;}.javascript .coMULTI {color: #006600; font-style: italic;}.javascript .es0 {color: #000099; font-weight: bold;}.javascript .br0 {color: #009900;}.javascript .sy0 {color: #339933;}.javascript .st0 {color: #3366CC;}.javascript .nu0 {color: #CC0000;}.javascript .me1 {color: #660066;}.javascript .ln-xtra, .javascript li.ln-xtra, .javascript div.ln-xtra {background:#FFFF88;}.javascript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/rnuatefB) - [See Original](https://pastebin.com/rnuatefB)</div>

1.  <div class="de1"><span class="kw1">var</span> j <span class="sy0">=</span> request<span class="sy0">;</span></div>

2.  <div class="de1"><span class="kw1">var</span> answer <span class="sy0">=</span> <span class="br0">{</span><span class="br0">}</span><span class="sy0">;</span> <span class="co1">//Prepare response payload</span></div>

4.  <div class="de1">    gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Starting process to add to Asset Movement Import Table'</span><span class="br0">)</span><span class="sy0">;</span></div>

5.  <div class="de2">    gs.<span class="me1">debug</span><span class="br0">(</span><span class="st0">'Script Include received payload: '</span> <span class="sy0">+</span> JSON.<span class="me1">stringify</span><span class="br0">(</span>j<span class="br0">)</span><span class="br0">)</span><span class="sy0">;</span></div>

</div>

Next we initialize a couple variables and start checking the data. Serial number is first because it's the most important:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.javascript .de1, .javascript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.javascript {color:#ACACAC;}.javascript .imp {font-weight: bold; color: red;}.javascript li, .javascript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.javascript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.javascript .kw1 {color: #000066; font-weight: bold;}.javascript .kw2 {color: #003366; font-weight: bold;}.javascript .kw3 {color: #000066;}.javascript .kw5 {color: #FF0000;}.javascript .co1 {color: #006600; font-style: italic;}.javascript .co2 {color: #009966; font-style: italic;}.javascript .coMULTI {color: #006600; font-style: italic;}.javascript .es0 {color: #000099; font-weight: bold;}.javascript .br0 {color: #009900;}.javascript .sy0 {color: #339933;}.javascript .st0 {color: #3366CC;}.javascript .nu0 {color: #CC0000;}.javascript .me1 {color: #660066;}.javascript .ln-xtra, .javascript li.ln-xtra, .javascript div.ln-xtra {background:#FFFF88;}.javascript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/XtFqfDcP) - [See Original](https://pastebin.com/XtFqfDcP)</div>

1.  <div class="de1"><span class="co1">//Initialize a couple variables before proceeding</span></div>

2.  <div class="de1"><span class="kw1">var</span> valid <span class="sy0">=</span> <span class="kw2">true</span><span class="sy0">;</span></div>

3.  <div class="de1"><span class="kw1">var</span> userValid <span class="sy0">=</span> <span class="kw2">false</span><span class="sy0">;</span></div>

5.  <div class="de2"><span class="co1">//Is Serial Number valid?</span></div>

6.  <div class="de1"><span class="kw1">var</span> assetGR <span class="sy0">=</span> <span class="kw1">new</span> GlideRecord<span class="br0">(</span><span class="st0">'alm_asset'</span><span class="br0">)</span><span class="sy0">;</span></div>

7.  <div class="de1">assetGR.<span class="me1">addQuery</span><span class="br0">(</span><span class="st0">'serial_number'</span><span class="sy0">,</span>j.<span class="me1">u_machine</span><span class="br0">)</span><span class="sy0">;</span></div>

8.  <div class="de1">assetGR.<span class="me1">query</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

9.  <div class="de1"><span class="kw1">if</span> <span class="br0">(</span>assetGR.<span class="me1">next</span><span class="br0">(</span><span class="br0">)</span><span class="br0">)</span><span class="br0">{</span></div>

10.  <div class="de2">    <span class="co1">//ok</span></div>

11.  <div class="de1"><span class="br0">}</span><span class="kw1">else</span><span class="br0">{</span></div>

12.  <div class="de1">    answer.<span class="me1">http_status</span> <span class="sy0">=</span> <span class="st0">"460"</span><span class="sy0">;</span></div>

13.  <div class="de1">    answer.<span class="me1">status_message</span> <span class="sy0">=</span> <span class="st0">'Rejected - Invalid Serial Number. Received '</span> <span class="sy0">+</span> j.<span class="me1">u_machine</span><span class="sy0">;</span></div>

15.  <div class="de2">    <span class="kw1">return</span> answer<span class="sy0">;</span></div>

16.  <div class="de1"><span class="br0">}</span></div>

</div>

Just a simple GlideRecord to make sure it exists is all we need. If it doesn't, we kick a Status = 460 back to Excel, which Excel interprets as a request to fix the Serial Number. After the Serial Number is verified, we look at the action code, but that's a boring run-on if statement. Much more interesting is the Assignment/User check:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.javascript .de1, .javascript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.javascript {color:#ACACAC;}.javascript .imp {font-weight: bold; color: red;}.javascript li, .javascript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.javascript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.javascript .kw1 {color: #000066; font-weight: bold;}.javascript .kw2 {color: #003366; font-weight: bold;}.javascript .kw3 {color: #000066;}.javascript .kw5 {color: #FF0000;}.javascript .co1 {color: #006600; font-style: italic;}.javascript .co2 {color: #009966; font-style: italic;}.javascript .coMULTI {color: #006600; font-style: italic;}.javascript .es0 {color: #000099; font-weight: bold;}.javascript .br0 {color: #009900;}.javascript .sy0 {color: #339933;}.javascript .st0 {color: #3366CC;}.javascript .nu0 {color: #CC0000;}.javascript .me1 {color: #660066;}.javascript .ln-xtra, .javascript li.ln-xtra, .javascript div.ln-xtra {background:#FFFF88;}.javascript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/28jpPyes) - [See Original](https://pastebin.com/28jpPyes)</div>

1.  <div class="de1"><span class="co1">//Is assignment a user?</span></div>

2.  <div class="de1"><span class="kw1">var</span> userGR <span class="sy0">=</span> <span class="kw1">new</span> GlideRecord<span class="br0">(</span><span class="st0">'sys_user'</span><span class="br0">)</span><span class="sy0">;</span></div>

3.  <div class="de1">userGR.<span class="me1">addQuery</span><span class="br0">(</span><span class="st0">'u_correlation_id'</span><span class="sy0">,</span> j.<span class="me1">u_assignment</span><span class="br0">)</span><span class="sy0">;</span></div>

4.  <div class="de1">userGR.<span class="me1">query</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

6.  <div class="de1"><span class="kw1">if</span> <span class="br0">(</span>userGR.<span class="me1">next</span><span class="br0">(</span><span class="br0">)</span><span class="br0">)</span><span class="br0">{</span></div>

7.  <div class="de1">    userValid <span class="sy0">=</span> <span class="kw2">true</span><span class="sy0">;</span></div>

8.  <div class="de1"><span class="br0">}</span></div>

10.  <div class="de2"><span class="co1">//Is assignment valid?</span></div>

11.  <div class="de1"><span class="kw1">if</span> <span class="br0">(</span>j.<span class="me1">u_assignment</span> <span class="sy0">==</span> <span class="st0">"NEW HARDWARE"</span> <span class="sy0">||</span> j.<span class="me1">u_assignment</span> <span class="sy0">==</span>  <span class="st0">"IMAGED"</span> <span class="sy0">||</span> j.<span class="me1">u_assignment</span> <span class="sy0">==</span>  <span class="st0">"TRIAGE"</span> <span class="sy0">||</span> j.<span class="me1">u_assignment</span> <span class="sy0">==</span>  <span class="st0">"DISPOSAL"</span> <span class="sy0">||</span> j.<span class="me1">u_assignment</span> <span class="sy0">==</span>  <span class="st0">"RMA"</span> <span class="sy0">||</span> userValid <span class="sy0">==</span> <span class="kw2">true</span><span class="br0">)</span><span class="br0">{</span></div>

12.  <div class="de1">    <span class="co1">//ok</span></div>

13.  <div class="de1"><span class="br0">}</span><span class="kw1">else</span><span class="br0">{</span></div>

14.  <div class="de1">    answer.<span class="me1">http_status</span> <span class="sy0">=</span> <span class="st0">"462"</span><span class="sy0">;</span></div>

15.  <div class="de2">    answer.<span class="me1">status_message</span> <span class="sy0">=</span> <span class="st0">'Rejected - Invalid Assignment. Received '</span> <span class="sy0">+</span> j.<span class="me1">u_assignment</span><span class="sy0">;</span></div>

17.  <div class="de1">    <span class="kw1">return</span> answer<span class="sy0">;</span></div>

18.  <div class="de1"><span class="br0">}</span></div>

</div>

First we use a GlideRecord to check whether the field contains a user's Correlation ID. This is a custom ID field that transcends all of our identity systems, over here. If it matches, it sets a bit, and then moves to check that bit + comparing the field to a few known Assignment Codes. If nothing matches, we kick back a Status = 462.

If all our checks pass, we push everything on to the next phase, then catch the RITM# and send it back to Excel for logging:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.javascript .de1, .javascript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.javascript {color:#ACACAC;}.javascript .imp {font-weight: bold; color: red;}.javascript li, .javascript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.javascript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.javascript .kw1 {color: #000066; font-weight: bold;}.javascript .kw2 {color: #003366; font-weight: bold;}.javascript .kw3 {color: #000066;}.javascript .kw5 {color: #FF0000;}.javascript .co1 {color: #006600; font-style: italic;}.javascript .co2 {color: #009966; font-style: italic;}.javascript .coMULTI {color: #006600; font-style: italic;}.javascript .es0 {color: #000099; font-weight: bold;}.javascript .br0 {color: #009900;}.javascript .sy0 {color: #339933;}.javascript .st0 {color: #3366CC;}.javascript .nu0 {color: #CC0000;}.javascript .me1 {color: #660066;}.javascript .ln-xtra, .javascript li.ln-xtra, .javascript div.ln-xtra {background:#FFFF88;}.javascript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/wAqgBpi7) - [See Original](https://pastebin.com/wAqgBpi7)</div>

1.  <div class="de1"><span class="co1">//If we're all good, send the data on to the Transform Map</span></div>

2.  <div class="de1"><span class="kw1">if</span><span class="br0">(</span>valid <span class="sy0">==</span> <span class="kw2">true</span><span class="br0">)</span><span class="br0">{</span></div>

3.  <div class="de1">    <span class="kw1">var</span> moveAsset <span class="sy0">=</span> <span class="kw1">new</span> GlideRecord<span class="br0">(</span><span class="st0">'x_ihgih_ssb_api_ssb_checkin_landing'</span><span class="br0">)</span><span class="sy0">;</span></div>

4.  <div class="de1">    moveAsset.<span class="me1">initialize</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

5.  <div class="de2">    moveAsset.<span class="me1">u_machine</span> <span class="sy0">=</span> j.<span class="me1">u_machine</span><span class="sy0">;</span></div>

6.  <div class="de1">    moveAsset.<span class="me1">u_action</span> <span class="sy0">=</span> j.<span class="me1">u_action</span><span class="sy0">;</span></div>

7.  <div class="de1">    moveAsset.<span class="me1">u_assignment</span> <span class="sy0">=</span> j.<span class="me1">u_assignment</span><span class="sy0">;</span></div>

8.  <div class="de1">    moveAsset.<span class="me1">u_stockroom</span> <span class="sy0">=</span> j.<span class="me1">u_stockroom</span><span class="sy0">;</span></div>

9.  <div class="de1">    moveAsset.<span class="me1">u_tech</span> <span class="sy0">=</span> j.<span class="me1">u_tech</span><span class="sy0">;</span></div>

11.  <div class="de1">    <span class="kw1">var</span> importSysId <span class="sy0">=</span> moveAsset.<span class="me1">insert</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

12.  <div class="de1">    <span class="kw1">var</span> importSetRowStatus <span class="sy0">=</span> <span class="st0">''</span><span class="sy0">;</span></div>

14.  <div class="de1">    <span class="kw1">var</span> assetRITM <span class="sy0">=</span> <span class="st0">'A'</span><span class="sy0">;</span></div>

15.  <div class="de2">    <span class="kw1">var</span> assetRITMgr <span class="sy0">=</span> <span class="kw1">new</span> GlideRecord<span class="br0">(</span><span class="st0">'sc_req_item'</span><span class="br0">)</span><span class="sy0">;</span></div>

16.  <div class="de1">    assetRITMgr.<span class="me1">addEncodedQuery</span><span class="br0">(</span><span class="st0">'cmdb_ci.serial_number='</span><span class="sy0">+</span>j.<span class="me1">u_machine</span><span class="sy0">+</span><span class="st0">'^ORDERBYDESCopened_at'</span><span class="br0">)</span><span class="sy0">;</span></div>

17.  <div class="de1">    assetRITMgr.<span class="me1">setLimit</span><span class="br0">(</span><span class="nu0">1</span><span class="br0">)</span><span class="sy0">;</span></div>

18.  <div class="de1">    assetRITMgr.<span class="me1">query</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

19.  <div class="de1">    <span class="kw1">if</span> <span class="br0">(</span>assetRITMgr.<span class="me1">next</span><span class="br0">(</span><span class="br0">)</span><span class="br0">)</span><span class="br0">{</span></div>

20.  <div class="de2">        assetRITM <span class="sy0">=</span> assetRITMgr.<span class="me1">number</span>.<span class="me1">toString</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

21.  <div class="de1">        gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Asset RITM gr: '</span> <span class="sy0">+</span> assetRITMgr<span class="br0">)</span><span class="sy0">;</span></div>

22.  <div class="de1">        gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Asset RITM: '</span> <span class="sy0">+</span> assetRITM<span class="br0">)</span><span class="sy0">;</span></div>

23.  <div class="de1">    <span class="br0">}</span></div>

25.  <div class="de2">    gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'No if Asset RITM: '</span> <span class="sy0">+</span> assetRITM<span class="br0">)</span><span class="sy0">;</span></div>

26.  <div class="de1">    answer.<span class="me1">http_status</span> <span class="sy0">=</span> <span class="st0">"200"</span><span class="sy0">;</span></div>

27.  <div class="de1">    answer.<span class="me1">status_message</span> <span class="sy0">=</span> <span class="st0">'All good. Request Number: '</span> <span class="sy0">+</span> assetRITM<span class="sy0">;</span></div>

28.  <div class="de1">    answer.<span class="me1">importSysId</span> <span class="sy0">=</span> importSysId<span class="sy0">;</span></div>

29.  <div class="de1">    answer.<span class="me1">RequestNum</span> <span class="sy0">=</span> assetRITM<span class="sy0">;</span></div>

30.  <div class="de2"><span class="br0">}</span></div>

31.  <div class="de1">gs.<span class="me1">debug</span><span class="br0">(</span><span class="st0">'Script Include assetVeritas concluding with: '</span> <span class="sy0">+</span> JSON.<span class="me1">stringify</span><span class="br0">(</span>answer<span class="br0">)</span><span class="br0">)</span><span class="sy0">;</span></div>

32.  <div class="de1"><span class="kw1">return</span> answer<span class="sy0">;</span></div>

</div>

## The Import Set and Transform Map

Version 1 of this implementation did not include the Scripted REST Endpoint or Script Include. Instead, the data came in from Excel and was written straight to an Import Table. his method was effective, but is probably not necessary if you're building this from scratch. What you will probably want, however, is the code below. In our instance, it runs as an onAfter script. First, it impersonates the Technician. This is important because we want the update to accurately reflect the Tech making it, and not "System Administrator" or some other such account. After that, we launch a Request. The Request's workflow is where we do the work. We did this because Requests already have a field for linking them with a CI, so it was an easy place to log the movements in perpetuity. This also allowed us to set up a Related List on the Asset Form that will list all of the recorded movements of the asset being shown.  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.javascript .de1, .javascript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.javascript {color:#ACACAC;}.javascript .imp {font-weight: bold; color: red;}.javascript li, .javascript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.javascript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.javascript .kw1 {color: #000066; font-weight: bold;}.javascript .kw2 {color: #003366; font-weight: bold;}.javascript .kw3 {color: #000066;}.javascript .kw5 {color: #FF0000;}.javascript .co1 {color: #006600; font-style: italic;}.javascript .co2 {color: #009966; font-style: italic;}.javascript .coMULTI {color: #006600; font-style: italic;}.javascript .es0 {color: #000099; font-weight: bold;}.javascript .br0 {color: #009900;}.javascript .sy0 {color: #339933;}.javascript .st0 {color: #3366CC;}.javascript .nu0 {color: #CC0000;}.javascript .me1 {color: #660066;}.javascript .ln-xtra, .javascript li.ln-xtra, .javascript div.ln-xtra {background:#FFFF88;}.javascript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/PV7NLvRH) - [See Original](https://pastebin.com/PV7NLvRH)</div>

1.  <div class="de1"><span class="kw1">if</span><span class="br0">(</span>req_submit <span class="sy0">==</span> <span class="kw2">true</span><span class="br0">)</span><span class="br0">{</span></div>

2.  <div class="de1">    <span class="co1">//Impersonate Tech - Imperative for Proper Audit reporting</span></div>

3.  <div class="de1">    gs.<span class="me1">include</span><span class="br0">(</span><span class="st0">'global.Impersonator'</span><span class="br0">)</span><span class="sy0">;</span></div>

4.  <div class="de1">    <span class="kw1">var</span> impersonator <span class="sy0">=</span>  <span class="kw1">new</span> global.<span class="me1">Impersonator</span><span class="br0">(</span><span class="br0">)</span>.<span class="me1">impersonateUser</span><span class="br0">(</span>techSYS<span class="br0">)</span><span class="sy0">;</span></div>

5.  <div class="de2">    gs.<span class="me1">debug</span><span class="br0">(</span><span class="st0">'I am impersonating '</span><span class="sy0">+</span>impersonator<span class="br0">)</span><span class="sy0">;</span></div>

6.  <div class="de1">    <span class="co1">//create RITM</span></div>

7.  <div class="de1">    <span class="kw1">var</span> cart <span class="sy0">=</span> <span class="kw1">new</span> sn_sc.<span class="me1">CartJS</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

8.  <div class="de1">    <span class="kw1">var</span> item <span class="sy0">=</span></div>

9.  <div class="de1">    <span class="br0">{</span></div>

10.  <div class="de2">        <span class="st0">'sysparm_id'</span><span class="sy0">:</span> <span class="st0">'27e0e16ddb505b0029c804c2ca96199b'</span><span class="sy0">,</span></div>

11.  <div class="de1">        <span class="st0">'sysparm_quantity'</span><span class="sy0">:</span> <span class="st0">'1'</span><span class="sy0">,</span></div>

12.  <div class="de1">        <span class="st0">'variables'</span><span class="sy0">:</span> <span class="br0">{</span></div>

13.  <div class="de1">            <span class="st0">'req_tech_ref'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>techSYS<span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

14.  <div class="de1">            <span class="st0">'u_requested_by'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>techSYS<span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

15.  <div class="de2">            <span class="st0">'req_machine_ref'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>target.<span class="me1">sys_id</span><span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

16.  <div class="de1">            <span class="st0">'req_action'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>source.<span class="me1">u_action</span><span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

17.  <div class="de1">            <span class="st0">'req_assignment_string'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>source.<span class="me1">u_assignment</span><span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

18.  <div class="de1">            <span class="st0">'req_assignment_ref'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>userSYS<span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

19.  <div class="de1">            <span class="st0">'u_requested_for'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>userSYS<span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

20.  <div class="de2">            <span class="st0">'req_imp_set'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>imp_set<span class="sy0">+</span><span class="st0">""</span><span class="sy0">,</span></div>

21.  <div class="de1">            <span class="st0">'req_stockroom'</span><span class="sy0">:</span> <span class="st0">""</span><span class="sy0">+</span>source.<span class="me1">u_stockroom</span><span class="sy0">+</span><span class="st0">""</span></div>

22.  <div class="de1">        <span class="br0">}</span><span class="br0">}</span><span class="sy0">;</span></div>

23.  <div class="de1">        <span class="kw1">var</span> cartDetails <span class="sy0">=</span> cart.<span class="me1">addToCart</span><span class="br0">(</span>item<span class="br0">)</span><span class="sy0">;</span></div>

24.  <div class="de1">        gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Cart details: '</span><span class="sy0">+</span>JSON.<span class="me1">stringify</span><span class="br0">(</span>cartDetails<span class="br0">)</span><span class="sy0">+</span><span class="st0">' Machine= '</span><span class="sy0">+</span>target.<span class="me1">sys_id</span><span class="br0">)</span><span class="sy0">;</span></div>

25.  <div class="de2">        gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Cart sysID: '</span><span class="sy0">+</span>cartDetails.<span class="me1">sys_id</span><span class="br0">)</span><span class="sy0">;</span></div>

26.  <div class="de1">        <span class="kw1">var</span> checkoutInfo <span class="sy0">=</span> cart.<span class="me1">checkoutCart</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

27.  <div class="de1">        gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Checkout Info: '</span><span class="sy0">+</span>checkoutInfo<span class="br0">)</span><span class="sy0">;</span></div>

28.  <div class="de1">        gs.<span class="me1">info</span><span class="br0">(</span><span class="st0">'Checkout Completed'</span><span class="br0">)</span><span class="sy0">;</span></div>

29.  <div class="de1"><span class="br0">}</span></div>

</div>

## The RITM and Workflow

I'm not really going to spend any time on the Requested Item, because, frankly, if you're still with me, you probably know enough to create an Item with the appropriate variables. So, we'll move on to the workflow:  
![Checkin Workflow](https://github.com/Greg-Malski/Asset-Checkin-Interface/raw/master/Screenshots/CheckinWorkflow.png)

Toney has written a dynamic subworkflow that we use to set some variables on intialization. If you bug him enough, he might showcase it here. Once that is finished, we move along to doing some work. I'm sure you can follow most of the branches, and some of the specific updates and checks are specific to our instance, so I won't go down those bunny trails. Instead, I'll just paste the script that makes the updates below and move on:  
<style>div.embedPastebin { text-align:left; padding: 0; color: #000; margin: 0; font-family: monospace; background: #F7F7F7; border: 1px solid ddd; border-radius:3px; } div.embedPastebin { } div.embedPastebin div.embedFooter { background: #F7F7F7; color: #333; font-size: 100%; padding: 6px 12px; border-bottom: 1px solid #ddd; text-transform:uppercase; } div.embedPastebin div.embedFooter a, div.embedPastebin div.embedFooter a:visited { color: #336699; text-decoration:none; } div.embedPastebin div.embedFooter a:hover { color: red; } .noLines ol { list-style-type: none; padding-left: 0.5em; } .embedPastebin{background-color:#F8F8F8;border:1px solid #ddd;font-size:12px;overflow:auto;margin: 0 0 0 0;padding:0 0 0 0;line-height:21px} .embedPastebin div { line-height:21px; font-family:Consolas, Menlo, Monaco, Lucida Console,'Bitstream Vera Sans Mono','Courier',monospace; } ol { margin:0; padding: 0 0 0 55px} ol li { border:0; margin:0;padding:0; } li.ln-xtra .de1, li.ln-xtra .de2 {background:#F8F8CE;} .embedPastebin ol li.li1 { margin: 0; } .embedPastebin ol li.li2 { margin: 0; }.javascript .de1, .javascript .de2 {-moz-user-select: text;-khtml-user-select: text;-webkit-user-select: text;-ms-user-select: text;user-select: text;margin:0; padding: 0 8px; background:none; vertical-align:top;color:#000;border-left: 1px solid #ddd; margin: 0 0 0 -7px; position: relative; background: #ffffff;}.javascript {color:#ACACAC;}.javascript .imp {font-weight: bold; color: red;}.javascript li, .javascript .li1 {-moz-user-select: -moz-none;-khtml-user-select: none;-webkit-user-select: none;-ms-user-select: none;user-select: none;}.javascript .ln {width:1px;text-align:right;margin:0;padding:0 2px;vertical-align:top;}.javascript .kw1 {color: #000066; font-weight: bold;}.javascript .kw2 {color: #003366; font-weight: bold;}.javascript .kw3 {color: #000066;}.javascript .kw5 {color: #FF0000;}.javascript .co1 {color: #006600; font-style: italic;}.javascript .co2 {color: #009966; font-style: italic;}.javascript .coMULTI {color: #006600; font-style: italic;}.javascript .es0 {color: #000099; font-weight: bold;}.javascript .br0 {color: #009900;}.javascript .sy0 {color: #339933;}.javascript .st0 {color: #3366CC;}.javascript .nu0 {color: #CC0000;}.javascript .me1 {color: #660066;}.javascript .ln-xtra, .javascript li.ln-xtra, .javascript div.ln-xtra {background:#FFFF88;}.javascript span.xtra { display:block; }</style>

<div class="embedPastebin">

<div class="embedFooter">Data hosted with ♥ by [Pastebin.com](https://pastebin.com) - [Download Raw](https://pastebin.com/raw/BGzE3qZC) - [See Original](https://pastebin.com/BGzE3qZC)</div>

1.  <div class="de1"><span class="kw1">var</span> target <span class="sy0">=</span> <span class="kw1">new</span> GlideRecord<span class="br0">(</span><span class="st0">'alm_asset'</span><span class="br0">)</span><span class="sy0">;</span></div>

3.  <div class="de1">target.<span class="me1">addQuery</span><span class="br0">(</span><span class="st0">'sys_id'</span><span class="sy0">,</span>current.<span class="me1">variable_pool</span>.<span class="me1">req_machine_ref</span><span class="br0">)</span><span class="sy0">;</span></div>

4.  <div class="de1">target.<span class="me1">query</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

5.  <div class="de2"><span class="kw1">if</span><span class="br0">(</span>target.<span class="me1">next</span><span class="br0">(</span><span class="br0">)</span><span class="br0">)</span><span class="br0">{</span></div>

6.  <div class="de1">    current.<span class="me1">cmdb_ci</span> <span class="sy0">=</span> target.<span class="me1">ci</span><span class="sy0">;</span> <span class="co1">//Associate request to CI</span></div>

7.  <div class="de1">    target.<span class="me1">assigned</span> <span class="sy0">=</span> current.<span class="me1">opened_at</span><span class="sy0">;</span> <span class="co1">//Set Assignment Date to match request</span></div>

9.  <div class="de1">    <span class="co1">//Set state for Check In/New Hardware</span></div>

10.  <div class="de2">    <span class="kw1">if</span> <span class="br0">(</span>current.<span class="me1">variable_pool</span>.<span class="me1">req_action</span> <span class="sy0">==</span> <span class="st0">'Check In'</span><span class="br0">)</span><span class="br0">{</span></div>

11.  <div class="de1">        <span class="kw1">if</span><span class="br0">(</span>current.<span class="me1">variable_pool</span>.<span class="me1">req_assignment_string</span> <span class="sy0">==</span> <span class="st0">'NEW HARDWARE'</span><span class="br0">)</span><span class="br0">{</span></div>

12.  <div class="de1">            target.<span class="me1">install_status</span> <span class="sy0">=</span> <span class="st0">'6'</span><span class="sy0">;</span> <span class="co1">//State = In Stock</span></div>

13.  <div class="de1">            target.<span class="me1">substatus</span> <span class="sy0">=</span> <span class="st0">'pending_image'</span><span class="sy0">;</span> <span class="co1">//Substate = Pending Image</span></div>

14.  <div class="de1">            target.<span class="me1">stockroom</span> <span class="sy0">=</span> current.<span class="me1">variable_pool</span>.<span class="me1">req_stockroom</span><span class="sy0">;</span> <span class="co1">//Stockrom matches submission</span></div>

15.  <div class="de2">            target.<span class="me1">update</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

16.  <div class="de1">            current.<span class="me1">short_description</span> <span class="sy0">=</span> <span class="st0">"Checked in, Pending Image"</span><span class="sy0">;</span></div>

17.  <div class="de1">        <span class="br0">}</span> <span class="kw1">else</span> <span class="kw1">if</span><span class="br0">(</span>current.<span class="me1">variable_pool</span>.<span class="me1">req_assignment_string</span> <span class="sy0">==</span> <span class="st0">'IMAGED'</span><span class="br0">)</span><span class="br0">{</span></div>

18.  <div class="de1">            target.<span class="me1">install_status</span> <span class="sy0">=</span> <span class="st0">'6'</span><span class="sy0">;</span> <span class="co1">//State = In Stock</span></div>

19.  <div class="de1">            target.<span class="me1">substatus</span> <span class="sy0">=</span> <span class="st0">'available'</span><span class="sy0">;</span> <span class="co1">//Substate = Available</span></div>

20.  <div class="de2">            target.<span class="me1">stockroom</span> <span class="sy0">=</span> current.<span class="me1">variable_pool</span>.<span class="me1">req_stockroom</span><span class="sy0">;</span> <span class="co1">//Stockrom matches submission</span></div>

21.  <div class="de1">            target.<span class="me1">update</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

22.  <div class="de1">            current.<span class="me1">short_description</span> <span class="sy0">=</span> <span class="st0">"Checked in after imaging"</span><span class="sy0">;</span></div>

23.  <div class="de1">        <span class="br0">}</span> <span class="kw1">else</span> <span class="kw1">if</span> <span class="br0">(</span>current.<span class="me1">variable_pool</span>.<span class="me1">req_assignment_string</span> <span class="sy0">==</span> <span class="st0">'TRIAGE'</span><span class="br0">)</span><span class="br0">{</span></div>

24.  <div class="de1">            target.<span class="me1">install_status</span> <span class="sy0">=</span> <span class="st0">'6'</span><span class="sy0">;</span> <span class="co1">//State = In Stock</span></div>

25.  <div class="de2">            target.<span class="me1">substatus</span> <span class="sy0">=</span> <span class="st0">'being_triaged'</span><span class="sy0">;</span> <span class="co1">//Substate = Being Triaged</span></div>

26.  <div class="de1">            target.<span class="me1">stockroom</span> <span class="sy0">=</span> current.<span class="me1">variable_pool</span>.<span class="me1">req_stockroom</span><span class="sy0">;</span> <span class="co1">//Stockrom matches submission</span></div>

27.  <div class="de1">            target.<span class="me1">update</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

28.  <div class="de1">            current.<span class="me1">short_description</span> <span class="sy0">=</span> <span class="st0">"Turned in, waiting for keep/dispose decision"</span><span class="sy0">;</span></div>

29.  <div class="de1">        <span class="br0">}</span></div>

30.  <div class="de2">    <span class="br0">}</span> <span class="kw1">else</span> <span class="kw1">if</span> <span class="br0">(</span>current.<span class="me1">variable_pool</span>.<span class="me1">req_action</span> <span class="sy0">==</span> <span class="st0">'Check Out'</span><span class="br0">)</span><span class="br0">{</span></div>

31.  <div class="de1">        <span class="kw1">if</span><span class="br0">(</span>current.<span class="me1">variable_pool</span>.<span class="me1">req_assignment_string</span> <span class="sy0">==</span> <span class="st0">'NEW HARDWARE'</span><span class="br0">)</span><span class="br0">{</span></div>

32.  <div class="de1">            target.<span class="me1">install_status</span> <span class="sy0">=</span> <span class="st0">'3'</span><span class="sy0">;</span> <span class="co1">//State = In Maintenance</span></div>

33.  <div class="de1">            target.<span class="me1">substatus</span> <span class="sy0">=</span> <span class="st0">'imaging'</span><span class="sy0">;</span> <span class="co1">//Substate = Imaging</span></div>

34.  <div class="de1">            target.<span class="me1">u_assigned_tech</span> <span class="sy0">=</span> current.<span class="me1">variable_pool</span>.<span class="me1">req_tech_ref</span><span class="sy0">;</span> <span class="co1">//Assign to Tech</span></div>

35.  <div class="de2">            target.<span class="me1">update</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">;</span></div>

36.  <div class="de1">            current.<span class="me1">short_description</span> <span class="sy0">=</span> <span class="st0">"Checked out to Tech for Imaging"</span><span class="sy0">;</span></div>

37.  <div class="de1">        <span class="br0">}</span></div>

38.  <div class="de1">    <span class="br0">}</span></div>

39.  <div class="de1"><span class="br0">}</span></div>

</div>

Ok, I cut it a little short because this post is already INSANELY long, but you get the idea. I'm going to wrap this up, here. You've got all the building blocks above, but I'll be posting all the code into GitHub so that you can see all the pieces in their natural state. Also, let me know if you have questions or suggestions.

[GitHub](https://github.com/Greg-Malski/Asset-Checkin-Interface)