The project's objective is to implement a web server that manages a static website composed of 3 HTML pages.

The HTTP server is based on a single socket listening on all network interfaces on port 8080. The server is based on the AF_INET protocol and the SOCK_STREAM type to ensure reliable TCP communication. At bind time, the SO_REUSEADDR option is enabled; this instructs the operating system to release the port immediately as soon as the process terminates, thus avoiding "Address already in use" errors during restarts.

When 'listen(1)' is called, the operating system prepares the socket to accept connections and creates a small queue to hold incoming requests. When a client connects, that request remains pending in the queue until we execute 'accept()'. At that point, 'accept()' removes the request from the queue and returns a new socket. This new socket is used exclusively for exchanging data with that specific client, while the original socket continues to listen on the same port, ready to receive other connections.

The server reads the initial data of the HTTP request from the socket and identifies the 'request-line', from which it extracts the method and the requested path. If the path is “/”, it is transformed into “/trails.html”. Otherwise, any “..” or leading slashes are removed to prevent navigation outside the “www” folder. Finally, we use commonpath to verify that the file is actually within “www”; if not, the request is blocked.

Once the file location is resolved, the server calculates the content-type using the mimetypes library, which maps the extension to a standard MIME type (.html, .jpeg, .css, etc.). Before sending the content, the HTTP header is constructed, including the status code (200 or 404), the Content-Type header with UTF-8 charset, and the Content-Length in bytes, followed by a double CRLF to separate the headers from the body. The body is transmitted in raw-binary mode using sendall(), ensuring the entire buffer is effectively delivered to the client.

Concurrently, every significant event—from the listener startup, connection acceptance, request-line details, to the response outcome and socket closure—is synchronously recorded to a log file with an ISO-style timestamp and severity level. This approach provides a comprehensive summary of all interactions, facilitating debugging and monitoring in production.

Once communication with the client is finished, we close the data exchange and release the socket address, so that other sockets may use it.
