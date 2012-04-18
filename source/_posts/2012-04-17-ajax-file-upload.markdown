---
layout: post
title: "AJAX upload on old browsers"
date: 2012-04-17 15:32
comments: true
share: true
footer: true
categories: 
- ajax 
- upload 
- jqform 
---

Web browsers supporting only XMLHTTPRequest Level 1 are not so uncommon and if you want to upload file using AJAX you should follow a few simple rules.

There is plenty of [sophisticated libraries](http://superdit.com/2010/06/29/10-jquery-ajax-file-uploader-plugins/) to manage ajax file upload. Being a fan of [jqForm](http://jquery.malsup.com/form/) that I massively use in the frontend of my applications, I decided to stick on that for this tut.

<!-- more -->

## Problem ##
Let's consider [the example](http://jquery.malsup.com/form/#file-upload) on the jqForm website. As explicitly explained, there are some limitations for old browsers when you need to get a JSON response back from server. 

> "Browsers that support the XMLHttpRequest Level 2 will be able to upload files seamlessly and even get progress updates as the upload proceeds. For older browsers, a fallback technology is used which involves iframes since it is not possible to upload files using the level 1 implmenentation of the XMLHttpRequest object. This is a common fallback technique, but it has inherent limitations. The iframe element is used as the target of the form's submit operation which means that the server response is written to the iframe. This is fine if the response type is HTML or XML, but doesn't work as well if the response type is script or JSON, both of which often contain characters that need to be repesented using entity references when found in HTML markup."


Official jqForm documentation suggests to modify the [server side code](http://jquery.malsup.com/form/files-raw.php) to let it send back the JSON data between a <code> &lt;textarea&gt; </code> tags.
I don't dislike their solution but I found an alternative that worked worked pretty well in a couple of my web apps and maybe it is simpler.

## Solution ##

### Server Side Code (Php) ###
``` php
<?php

// process uploaded file

$data = array();

// populate $data with the json data to return

echo json_encode($data);
?>
```

As you can see, I have voluntarily not specified the response Content-Type to workaround the problem we are addressing.

### Client Side Code (Javascript) ####
``` javascript
<script type="text/Javascript">
 // I'm assuming that the form id is "myUploadForm"
 $('#myUploadForm').ajaxForm({
        beforeSubmit: function(a,f,o) {
    		// show loader gif
        },
        success: function(data) {
        	// here we get raw data, not yet encoded in JSON object
        	var json_data = null;
        	try{
        		json_data = jQuery.parseJSON(data);
        		// make something with json_data
        	}
        	catch(e)
        	{
    			alert('Ops, something went wrong :(');
        	}
        },
        complete: function()
        {
        	// hide loader gif
        },
        error: function(jqXHR, textStatus, errorThrown)
        {
        	alert('Ops, something went wrong :(');
        }
    });
</script>
```

Don't set the Content-Type in the server response and let jQuery intelligently guess the response content type ( take a look at [datatype](http://api.jquery.com/jQuery.ajax/) option specifications ).