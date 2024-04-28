# Web Form Tutorial

A quick tutorial on how to access a remote resource and dynamically update a web page with javascript.

- [Requirements](#requirements)
- [Setup](#setup)
- [CGI Tutorial](#cgi-tutorial)
- [Javascript Integration](#javascript-integration)


## Requirements [^](#web-form-tutorial)

* [Docker](https://www.docker.com)
* [Python 3](https://www.python.org)


## Setup [^](#web-form-tutorial)

* Echo Server Container
  
    An echo server responds with data sent to the server. This echo server includes additional request information and can modify the response headers. This tutorial uses the echo server as as an example remote resource to call.



    ```
    docker run -d -p 3000:80 ealen/echo-server
    ```

    https://hub.docker.com/r/ealen/echo-server

    <details>
      <summary>Stopping and removing the container</summary>

    ```
    echo_server_container=$(docker ps | grep echo-server | awk '{print $1}'); docker stop $echo_server_container; docker remove $echo_server_container; unset echo_server_container
    ```
    </details>


## CGI tutorial [^](#web-form-tutorial)


### Command line

* curl

    A command line tool to make http requests. Use `curl` to fetch the output of a resource request. This provides a chance to view the output schema for later reference.

    request:

    ```
    curl http://localhost:3000
    ```

    response:

    ``` no-copy
    {
        "host":{
            "hostname":"localhost",
            "ip":"::ffff:192.168.65.1",
            "ips":[]
        },
        "http":{
            "method":"GET",
            "baseUrl":"",
            "originalUrl":"/",
            "protocol":"http"
        },
        "request":{
            "params":{
                "0":"/"
            },
            "query":{},
            "cookies":{},
            "body":{},
            "headers":{
                "host":"localhost:3000",
                "user-agent":"curl/8.4.0",
                "accept":"*/*"
            }
        },
        "environment":{
            "PATH":"/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "HOSTNAME":"541e7ab2f3bb",
            "NODE_VERSION":"20.11.0",
            "YARN_VERSION":"1.22.19",
            "HOME":"/root"
        }
    }
    ```

  <details>
    <summary>Additional CGI Requests</summary>

    * Fetch data from URL

        ```
        curl http://localhost:3000/resource
        ```

    * HTTP Methods

        ```
        curl http://localhost:3000/resource -X GET
        ```

    * HTTP Query Parameters

        ```
        curl http://localhost:3000/resource?Param0=Value0&Param1=Value1 -X GET
        ```

    * HTTP Request Body

        ```
        curl http://localhost:3000/resource -X POST -d '{"name0": "value0", "name1": "value1"}'
        ```

    * HTTP JSON Request Body

        ```
        curl http://localhost:3000/resource -H 'Content-Type: application/json' -X POST -d '{"key0": "value0", "key1": "value1"}'
        ```
  </details>


### Web Form

* example form code

    This base html code provides example CGI form GET and POST calls to a remote resource.

    ```
    <html>
    <head>
        <title>Form Primer</title>
    </head>
    <body>
        <a href="http://localhost:3000/">test GET</a><br />
        <br />
        <a href="http://localhost:3000/resource">test GET Resource</a><br />
        <br />
        <form action="http://localhost:3000/" method="post">
            <input type="text" name="name" placeholder="Name">
            <input type="text" name="email" placeholder="Email">
            <input type="submit" name="submit" value="Submit">
        </form>
    </body>
    </html>
    ```


## Javascript Integration [^](#web-form-tutorial)

* Start a python web server

    In the same directory as the index.html file run:

    ```
    python -m http.server
    ```

    This starts a python webserver and serves files from the directory the command is run in.

    <details>
      <summary>Stopping the Webserver</summary>
      type `<ctrl-C>` to exit the server
    </details>


* Form Code

    ```
    <!DOCTYPE HTML>
    <html>
    <head>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
        <title>Form Primer</title>
    </head>
    <body>
        <div class="vstack gap-3">
            <div class="p-2">
                <a href="http://localhost:3000/">test GET</a><br />
            </div>
            <div class="p-2">
                <a href="http://localhost:3000/resource">test GET Resource</a><br />
            </div>
            <form action="http://localhost:3000/" method="post">
                <div class="hstack p-2 gap-3">
                    <div>
                        <input type="text" class="form-control" name="name" placeholder="Name">
                    </div>
                    <div>
                        <input type="text" class="form-control" name="email" placeholder="Email">
                    </div>
                    <button type="submit" name="submit" class="btn btn-primary">Submit</button>
                </div>
            </form>
            <div class="hstack p-2 gap-3">
                <button class="btn btn-primary" id="fetchDataButton">Fetch Data</button>
                <button class="btn btn-primary" id="clearButton">Clear Output</button>
            </div>
            <pre id="output"></pre>
        </div>
    </body>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz" crossorigin="anonymous"></script>
    <script>
        async function fetchResource() {
            let response;
            try {
                const url = 'http://localhost:3000';
                const headers = new Headers({
                    "Content-Type": "application/json",
                    "X-ECHO-HEADER": "Access-Control-Allow-Origin:*, Access-Control-Allow-Headers:*",
                });
              
                let body = {
                    name: 'test',
                    email: 'user@exmaple.com'
                };

                response = await fetch(url, {
                    method: 'POST',
                    headers: headers,
                    body: JSON.stringify(body)
                });
            }
            catch (e) {
                console.error(e);
                throw e;
            }

            if (response?.ok) {
                responseJson = await response.json();
                let output = document.getElementById('output');
                output.innerHTML = JSON.stringify(responseJson, null, 2);
            }
        }
        let fetchDataButton = document.getElementById('fetchDataButton');
        fetchDataButton.addEventListener('click', fetchResource);

        async function clearOutput() {
            let output = document.getElementById('output');
            output.innerHTML = '';
        }
        let clearButton = document.getElementById('clearButton');
        clearButton.addEventListener('click', clearOutput);

    </script>
    </html>
    ```
