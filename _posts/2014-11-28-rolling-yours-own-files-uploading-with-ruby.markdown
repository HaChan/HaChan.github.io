---
layout: post
title:  "Rolling yours owns files uploading via POST using Net:HTTP"
date:   2014-11-28
tags: [ruby]
---

First, you'll need to know how the browser work on uploading files or how the HTTP request looks like when sending as upload files request.

To upload files in the browser, we use a form like this:

```html
<form enctype="multipart/form-data" action="http://localhost:3000/" method="POST">
  <input type="hidden" name="MAX_FILE_SIZE" value="100000" />
  Choose a file to upload: <input name="uploadedfile" type="file" /><br />
  <input type="submit" value="Upload File" />
</form>
```

The browser will construct a _multipart_ HTTP request message and send it to the webserver:

```
POST / HTTP/1.1
Host: localhost:3000
Content-Length: 1325
Origin: http://localhost:3000
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryePkpFF7tjBAqx29L
.
.
.
<other headers>

------WebKitFormBoundaryePkpFF7tjBAqx29L
Content-Disposition: form-data; name="MAX_FILE_SIZE"

100000
------WebKitFormBoundaryePkpFF7tjBAqx29L
Content-Disposition: form-data; name="uploadedfile"; filename="image.png"
Content-Type: image/png

<file data (image.png binary)>
------WebKitFormBoundaryePkpFF7tjBAqx29L--
```

Instead of URL encoding the form parameters, the form parameters (including the file data) are sent as sections in a multipart document in the body of the request.

As you can see in the above example, the multipart HTTP request message consists of a number of sections separated by a boundary=

    ----WebKitFormBoundaryePkpFF7tjBAqx29L

Each section represent a set value in the form (form field) and it contains a number of headers, a `\r\n`, content (form field values) and finishes with a `\r\n`. So, the general normal field will look like this:

```
Content-Disposition: form-data; name="form-field"

form-field-value
```

And for the file filed, it will look like this:

```
Content-Disposition: form-data; name="fieldname"; filename="filename"
Content-Type: file-mime-type

BINARYDATA...
```

The request body will be terminated by `boundary + "--"`.

The Net::HTTP ruby library only accepts raw content, which mean it will only accepts a string HTTP request. So, in order to trigger a post upload file request, we must build the request string message manually.

Let's build the multipart sections first. There are two type of sections: one is normal and the other is file part. We will need two method for each section:

_Normal_ section

```ruby
def multipart_text key, value
  content = "Content-Disposition: form-data; name=\"#{key}\"" <<
    "\r\n" << "\r\n" << "#{value}" << "\r\n"
end
```

_File_ section

```ruby
def multipart_file key, filename, mime_type, content
  content = "Content-Disposition: form-data; name=\"#{key}\"; filename=\"#{filename}\"#{"\r\n"}" <<
    "Content-Type: #{mime_type}\r\n" << "\r\n" << "#{content}" << "\r\n"
end
```

These two method is encapsulate in a class called MultipartPost, which I will used to make the multipart request body and issuse a request to a web server. This class will have an array attribute to hold all the sections of the request and an attribute to hold the request url.

```ruby
class MultipartPost
  EOL = "\r\n"

  def initialize uri, &block
    @params = Array.new
    @uri = URI.parse uri
    instance_eval &block if block
  end

  private
  def multipart_text key, value
    content = "Content-Disposition: form-data; name=\"#{key}\"" <<
      EOL << EOL << "#{value}" << EOL
  end

  def multipart_file key, filename, mime_type, content
    content = "Content-Disposition: form-data; name=\"#{key}\"; filename=\"#{filename}\"#{EOL}" <<
      "Content-Type: #{mime_type}\r\n" << EOL << "#{content}" << EOL
  end
end
```

To add sections to the @params, I will need two addition methods:

```ruby
class MultipartPost
  # omited codes...

  def params_part key, value
    @params << multipart_text(key, value)
  end

  def files_part key, filename, mime_type, content
    @params << multipart_file(key, filename, mime_type, content)
  end

  private
  # omited codes...
end
```

Now, I have to put all the section into one string. Remeber that each section will be separated by a boundary and terminated with `boundary + "--"`. So, this class has to have a method to putting it altogether:

```ruby
class MultipartPost
  BOUNDARY = "-----------RubyMultipartPost"

  # omited codes...
  def request_body
    body = @params.map{|p| "--#{BOUNDARY}#{EOL}" << p}.join ""
    body << "#{EOL}--#{BOUNDARY}--#{EOL}"
  end

  # omited codes...
end
```

Now, I have everything I need to create a multipart post request string for upload files. The only left is issue the request to a web server. This is the job for ruby Net:HTTP library. The MultipartPost class need a method that use Net::HTTP library to issuse the request, like so:

```ruby
def run
  http = Net::HTTP.new @uri.host, @uri.port
  request = Net::HTTP::Post.new @uri.request_uri
  request.body = request_body
  request.set_content_type "multipart/form-data", {"boundary" => BOUNDARY}
  res = http.request request
  res.body
end
```

Using this class is very simple. All you need to do is create a new MultipartPost class instance with an url string and passed it a block that constructing the section, and then call `run()` method on that instance.

```ruby
multi_part = MultipartPost.new post_url do
  params_part "key", value
  files_part "file-key", filename,
    file_content_type, file-data
end
multi_part.run
```

My full source code [here](https://gist.github.com/HaChan/fd7c8d5e1a07b54e472c#file-multipart_post-rb)
