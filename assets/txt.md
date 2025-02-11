import os
from http.server import HTTPServer, SimpleHTTPRequestHandler

class MyHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        path = self.translate_path(self.path)  # Use translate_path for security
        if os.path.isdir(path):
            # Serve directory listing if it's a directory
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            self.wfile.write(self.list_directory(path).encode())  # Encode to bytes
        elif os.path.isfile(path):
            # Serve file if it's a file
            f = open(path, 'rb')  # Open in binary mode
            self.send_response(200)
            self.send_header('Content-type', self.guess_type(path))
            self.send_header('Content-Length', str(os.path.getsize(path)))
            self.end_headers()
            self.wfile.write(f.read())
            f.close()
        else:
            # Handle cases where the path is neither a file nor a directory
            self.send_error(404, "File or directory not found")

    def list_directory(self, path):
        """Generate a directory listing HTML."""
        try:
            list = os.listdir(path)
        except os.error:
            return self.send_error(404, "No permission to list directory")
        list.sort()
        r = []
        displaypath = cgi.escape(urllib.parse.unquote(self.path))
        r.append(f'<title>Directory listing for {displaypath}</title>\n')
        r.append(f'<h2>Directory listing for {displaypath}</h2>\n')
        r.append('<hr>\n<ul>\n')
        for name in list:
            fullname = os.path.join(path, name)
            displayname = name + "/" if os.path.isdir(fullname) else name
            linkname = name + "/" if os.path.isdir(fullname) else name
            r.append(f'<li><a href="{linkname}">{displayname}</a></li>\n')
        r.append('</ul>\n<hr>\n')
        return "".join(r)


if __name__ == '__main__':
    import cgi, urllib.parse
    Handler = MyHandler
    httpd = HTTPServer(("", 8000), Handler) # Port 8000
    print("Serving at port 8000")
    httpd.serve_forever()
