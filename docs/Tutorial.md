## Example: Streamed Picture Upload

In this tutorial, we create a web page that accepts single picture uploads, converts the picture to data url, and generates HTML to display it. Everything will be done in memory, no temporary files will be created.

Complete source code is available here: [streamed_upload_pic.cpp](https://github.com/ReimuNotMoe/Marisa/blob/master/Source/Tests/streamed_upload_pic.cpp)

First the includes:

    #include <iostream> // For std::cout
    #include <Marisa.hpp>
    
Import things in these namespaces to make things easier:

    using namespace Marisa::Application;
    using namespace Middlewares;
    
Let's create a middleware class to handle the uploads:

    class upload_handler : public Middleware {
    
    public:
    
        void handler() override;
        std::unique_ptr<Middleware> New() const override;
        
    }

The class will automatically inherit these variables:

- `RequestContext *request`: Contains information about the client request, and functions to manipulate them.
- `ResponseContext *response`: Contains information about the response we want to send back to client, and functions to manipulate them.
- `ContextExposed *context`: Contains internal data & states of the current route, middlewares, and I/O. Normally we don't need to touch them.

First, we have to take care of the `New()` method. Its main purpose is to create a new instance of this class in downcasted version for the framework, with no data loss.

We also need to copy or `make_shared()` any variables that were passed to the class constructor, if there are any. Now we have none, so we can just use the default constructor.

    std::unique_ptr<Middleware> New() const override {
		return std::make_unique<upload_handler>();
	}

And then we need to do our main logic in the `handler()` method.

    void handler() override {
    
First, let's create a multipart parser. It's called `Mullet`.

<s>I have no idea why I wanted to give it this name. Maybe because of express's `Multer`?</s>

<s>Tbh, I actually looked through a dictionary and tried to find a mul- word that looks good to me. :P</s>

    Mullet mp;

Then, let it parse the lowercased headers (since we won't be able to know whether the client will send stuff like `Content-Type` or `content-type` or even `cOnTEnT-tYPE`).

It will automatically determine the content type and part boundary in this process.

    mp.set_headers(request->headers_lowercase);
    
Create a base64 encoder for the data url, and a string to hold the encoded data.

Due to limitations of HTTP, we can't send data until the client's request is fully received. Otherwise we can stream the encoded data back!

    Base64::Encoder b64_enc;
    std::string encoded;

And a loop for the process:

    while (true) {
    
Read only a part of client's POST data (max 8KB here), and let Mullet parse it.

`request->read()` will return a `std::vector<uint8_t>`.

    auto buf = request->read(8192);
    std::cout << "read " << buf.size() << " bytes\n";
    mp.parse(buf);
