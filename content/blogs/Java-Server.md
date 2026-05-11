---
title: "Java Server"
date: 2025-10-15
draft: false
tags: ["java", "server", "socket", "api"]
summary: "Creating a Java server from scratch using socket programming and working with APIs."
---


## In this blog, I would:

1. [Create a Hello world Java server](#first-we-create-a-hello-world-java-server)
2. [Use NASA APOD API](#now-we-extend-the-server-to-display-pictures-from-the-nasa-adop-api)
3. [User jokeapi.dev to tell you jokes](#integrate-jokeapi-to-get-jokes)
4. [And get 2 servers to talk to each other. A basic Chat application.](#now-lets-try-running-2-you-can-run-more-servers-on-the-same-port-and-make-them-talk-to-each-other)


This blog is to show how people can work with simple APIs and socket using Java, and create a java server
from scratch.

### First we create a Hello world Java server.

```java
public class SimpleHttpServer {
    public static void main(String[] args) {
        int port = 8080; // You can change this to any available port

        try {
            ServerSocket serverSocket = new ServerSocket(port);
            System.out.println("Server listening on port " + port);

            while (true) {
                Socket clientSocket = serverSocket.accept();
                handleRequest(clientSocket);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void handleRequest(Socket clientSocket) throws IOException {
        // try-with-resources statement introduced in Java 7
        try (
                Scanner in = new Scanner(clientSocket.getInputStream());
                PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {
            // Read the HTTP request from the client
            while (in.hasNextLine()) {
                String line = in.nextLine();
                System.out.println("Client request: " + line);

                // Break when an empty line is encountered, signaling the end of the request
                if (line.isEmpty()) {
                    break;
                }
            }

            // Send a simple HTTP response back to the client
            String response = "HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\n\r\nHello World!, this is a server!";
            out.println(response);
        }
        // Close the client socket
        clientSocket.close();
    }
}
```

#### Now, a small discussion for every module used in this code,

* IOException:
   This module is used to handle Input/Output (IO) exceptions. In the context of the server,
   IO operations like reading from or writing to streams can throw IOExceptions.

* PrintWriter:
   This module provides the capability to print formatted representations of objects to a
   text-output stream.

* ServerSocket:
   This module provides a mechanism to listen for incoming connections on a specific port.

* Socket:
   This module represents a socket, which is an endpoint for sending or receiving data across
   a computer network. In the server code, Socket is used to accept connections from clients.

* Scanner:
   This module is used for parsing primitive types and strings using regular expressions.

![Hello World](./static/blogs/Java-Server/HelloWorld.png)

- You can see the requests getting logged in the terminal.
  Now we can only send "GET" requests from the browser.

### Now we extend the server, to display pictures from the NASA ADOP API.

API_KEY:`DEMO_KEY`
API:`https://api.nasa.gov/planetary/apod?api_key=`

```java
public class NasaADOP {
    public static void main(String[] args) {
        int port = 8080;

        try {
            ServerSocket serverSocket = new ServerSocket(port);
            System.out.println("Server listening on port " + port);

            while (true) {
                Socket clientSocket = serverSocket.accept();
                handleRequest(clientSocket);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void handleRequest(Socket clientSocket) {
        try (
                Scanner in = new Scanner(clientSocket.getInputStream());
                PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {
            // Read the HTTP request from the client
            while (in.hasNextLine()) {
                String line = in.nextLine();
                System.out.println("Client request: " + line);

                // Break when an empty line is encountered, signaling the end of the request
                if (line.isEmpty()) {
                    break;
                }
            }

            // Fetch image URL from NASA API
            String[] imageUrlAndDescription = getImageUrlFromNasaApi();
            String embed = buildHtmlResponse(imageUrlAndDescription);
            StringBuilder response = new StringBuilder();

response.append("HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n");
            response.append("<html><body><h2>Hello World!, this is today's APOD!</h2>");
            response.append(embed);
            response.append("</body></html>");
            out.println(response.toString());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // Close the client socket
            try {
                clientSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private static String[] getImageUrlFromNasaApi() {
        // Make an HTTP request to the NASA API to get the image URL
        // Replace this with your actual NASA API key and the correct API endpoint
        String apiKey = "DEMO_KEY";
        String apiUrl = "https://api.nasa.gov/planetary/apod?api_key=" + apiKey;
        String[] returnStrings = new String[3];
        try (Scanner scanner = new Scanner(new URL(apiUrl).openStream())) {
            StringBuilder response = new StringBuilder();
            while (scanner.hasNextLine()) {
                response.append(scanner.nextLine());
            }

            // Parse the JSON response to get the image / video URL
            // This is a simplified example, you may want to use a JSON
            // library for robust parsing
            String responseString = response.toString();
            String title = responseString.split("title\":\"")[1].split("\",")[0];
            String mediaLink = responseString.split("\"url\":\"")[1].split("\"}")[0];
            String explanation = responseString.split("explanation\":\"")[1].split("\",")[0];

            if (mediaLink.length() > 2) {
                returnStrings[0] = title;
                returnStrings[1] = mediaLink;
                returnStrings[2] = explanation;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return returnStrings;
    }

    private static String buildHtmlResponse(String[] imageUrlAndDescription) {
        StringBuilder embed = new StringBuilder();
        embed.append("<p><h3>"+imageUrlAndDescription[0]+"</h3></p>");
        embed.append("<p><img src=\""+imageUrlAndDescription[1]+"\"></p>");
        embed.append("<p>"+imageUrlAndDescription[2]+"</p>");

        return embed.toString();
    }
}

```

![APOD Sample](./static/blogs/Java-Server/APOD.png)

Now, a small discussion for other module used in this code,

* URL:
  the URL class in Java provides a convenient and straightforward way to work with URLs,
  making it a valuable tool for tasks involving network resources and web services.

###  Integrate Joke.api to get jokes:

API: `https://v2.jokeapi.dev/joke/Any`

```java
public class GetThisJoke {
    public static void main(String[] args) {
        int port = 8080;

        try {
            ServerSocket serverSocket = new ServerSocket(port);
            System.out.println("Server listening on port " + port);

            while (true) {
                Socket clientSocket = serverSocket.accept();
                handleRequest(clientSocket);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void handleRequest(Socket clientSocket) {
        try (
                Scanner in = new Scanner(clientSocket.getInputStream());
                PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {
            // Read the HTTP request from the client
            while (in.hasNextLine()) {
                String line = in.nextLine();
                System.out.println("Client request: " + line);

                // Break when an empty line is encountered, signaling the end of the request
                if (line.isEmpty()) {
                    break;
                }
            }

            // Send an HTML response with a form
            String[] joke = makeApiRequest();
            StringBuilder outJoke = new StringBuilder();
            if (joke[1].length() > 1) {
                outJoke.append(joke[0]);
                outJoke.append("\n\r");
                outJoke.append(joke[1]);
            } else {
                outJoke.append(joke[0]);
            }
            String embed = outJoke
                    .toString()
                    .replace("\\n", " ")
                    .replace("\\\"", "");

                    StringBuilder response = new StringBuilder();
            response.append("HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n");
            response.append("<html><body>");
            response.append("<h2>Hello World!, this is a server with Java backend!</h2>");
            response.append("<form action=\"/submit\" method=\"get\">");
            response.append("<input type=\"submit\" value=\"Get new joke!\"></form>");
            response.append("<p id=\"apiResponse\">" + embed + "</p></body></html>");
            out.println(response.toString());

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // Close the client socket
            try {
                clientSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private static String[] makeApiRequest() {
        // Simulate making an API request and return the response
        String apiUrl = "https://v2.jokeapi.dev/joke/Any";
        try (
                Scanner scanner = new Scanner(new URL(apiUrl).openStream())) {
            StringBuilder response = new StringBuilder();

            while (scanner.hasNextLine()) {
                response.append(scanner.nextLine());
            }

            String responseString = response.toString();
            if (responseString.contains("\"joke\"")) {
                String premise = responseString.split("\"joke\":")[1].split("\"flags\"")[0];
                String[] returnResponse = { premise, new String() };
                return returnResponse;
            } else {
                String premise = responseString.split("\"setup\":")[1].split("\"delivery\"")[0];
                String punchline = responseString.split("\"delivery\":")[1].split("\"flags\"")[0];
                return new String[] { premise, punchline };
            }
        } catch (IOException ioException) {
            ioException.printStackTrace();
            ;
        }
        String[] empStrings = { "", "" };
        return empStrings;
    }
}

```

![Sample joke](./static/blogs/Java-Server/joke.png)


### Now, lets try running 2 (you can run more) servers on the same port and make them talk to each other

Server 1:
```java
public class Server1 {
    public static void main(String[] args) {
        try {
            int port = 8080;
            ServerSocket serverSocket = new ServerSocket(port);
            System.out.println("Server 1 is waiting for connections...");

            Socket socket = serverSocket.accept();
            System.out.println("Connected to client: " + socket.getInetAddress());

            BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter writer = new PrintWriter(socket.getOutputStream(), true);

            new Thread(() -> {
                try {
                    while (true) {
                        // Server 1 receives a message
                        String receivedMessage = reader.readLine();
                        if (receivedMessage == null) {
                            break;
                        }
                        System.out.println("Received: " + receivedMessage);

                        // Server 1 sends a response
                        System.out.print("Server 1: ");
                        String response = getUserInput();
                        writer.println(response);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();

            while (true) {
                // Server 1 sends a message
                System.out.print("Server 1: ");
                String message = getUserInput();
                writer.println(message);

                // Server 1 receives a response
                String response = reader.readLine();
                System.out.println("Received: " + response);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static String getUserInput() {
        try {
            BufferedReader consoleReader = new BufferedReader(new InputStreamReader(System.in));
            return consoleReader.readLine();
        } catch (IOException e) {
            e.printStackTrace();
            return "";
        }
    }
}
```
Server 2 code is same with minor differences in print statements.
When you run them:

![Server Terminal](./static/blogs/Java-Server/server.png)
![Client Terminal](./static/blogs/Java-Server/client.png)

* BufferedReader: Used for reading text from a character-based input stream (e.g., reading messages from the client).
* InputStreamReader: Converts bytes into characters. In this context, it's used to wrap the InputStream from the Socket to create a BufferedReader for reading messages.

5. Now lets try running servers on different ports and make then talk to each other.

References:

1. [java.net.URL](https://www.baeldung.com/java-url)
2. [java.net](https://docs.oracle.com/javase/8/docs/api/java/net/package-summary.html)
3. [NASA APIs](https://api.nasa.gov/)
4. [Java chat server](https://medium.com/nerd-for-tech/create-a-chat-app-with-java-sockets-8449fdaa933)