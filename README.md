UDP Client/Server

Overview

This program is a small Python UDP client/server application designed to demonstrate basic network programming with Python's "socket" module.

The program can operate in two roles:

* "server" — listens for UDP datagrams and responds with the size of the received data.
* "client" — sends the current date and time to the server and displays the server's response.

The communication takes place locally over the IPv4 loopback interface.

How It Works

The server binds to a specific IP address and UDP port:

.. code-block:: text

127.0.0.1:1027

The client does not explicitly bind to a local address or port. When it sends its first UDP datagram, the operating system automatically assigns it an ephemeral local port.

A typical communication therefore looks like:

.. code-block:: text

Client                         Server
127.0.0.1:41879  ──────────→  127.0.0.1:1027
                   UDP
                 datagram

Client                         Server
127.0.0.1:41879  ←──────────  127.0.0.1:1027
                   UDP
                 response

Requirements

* Python 3
* Python standard library

No third-party Python packages are required.

Running the Program

Start the server:

.. code-block:: console

python udp.py server

The default port is "1027".

A different port can be specified with "-p":

.. code-block:: console

python udp.py server -p 5000

Then, in another terminal, start the client:

.. code-block:: console

python udp.py client

If the server is using a different port:

.. code-block:: console

python udp.py client -p 5000

Command-Line Arguments

The program uses Python's "argparse" module.

"role"


A required positional argument specifying whether the program should operate as a client or server.

Valid values:

* ``client``
* ``server``

``-p PORT``

Optional UDP port.

Default:

.. code-block:: text

1027

Examples:

.. code-block:: console

python udp.py server
python udp.py client

python udp.py server -p 5000
python udp.py client -p 5000

Server Operation

The server creates an IPv4 UDP socket:

.. code-block:: python

socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

"AF_INET" specifies IPv4.

"SOCK_DGRAM" specifies a datagram socket, which uses UDP.

The server then explicitly binds the socket:

.. code-block:: python

sock.bind(('127.0.0.1', port))

This gives the server a known local endpoint.

It then waits for incoming UDP datagrams:

.. code-block:: python

data, address = sock.recvfrom(MAX_BYTE)

"recvfrom()" returns two values:

* "data" — the received bytes.
* "address" — the sender's IP address and port.

The server prints the sender and received data, then calculates the number of bytes received:

.. code-block:: python

len(data)

It sends the response back to the address returned by "recvfrom()":

.. code-block:: python

sock.sendto(data, address)

The server runs inside a "while True" loop so that it can continue receiving datagrams instead of terminating after the first client request.

Client Operation

The client also creates an IPv4 UDP socket:

.. code-block:: python

socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

Unlike the server, the client does not explicitly call "bind()".

It creates a message containing the current date and time:

.. code-block:: python

text = f'The time is {datetime.now()}'

Because sockets transmit bytes, the string is encoded:

.. code-block:: python

data = text.encode('ascii')

The client then sends the datagram to the server:

.. code-block:: python

sock.sendto(data, ('127.0.0.1', port))

Because the client did not specify a local port, the operating system automatically assigns an ephemeral port.

For example:

.. code-block:: text

('127.0.0.1', 41879)

The client can inspect its local socket address using:

.. code-block:: python

sock.getsockname()

It then waits for the server's response:

.. code-block:: python

data, address = sock.recvfrom(MAX_BYTE)

Finally, it decodes the received bytes back into text.

Data Representation

The socket API operates on bytes.

The client therefore converts its Python string into bytes before transmission:

.. code-block:: text

Python string
      |
      | encode()
      v
    bytes
      |
      | sendto()
      v
   network

The server receives bytes and can convert them back to text with "decode()".

The "!r" conversion used in the output displays the representation of the value, making byte data and escape characters visible.

For example:

.. code-block:: python

print(f'{data!r}')

may display:

.. code-block:: text

b'The time is 2026-10-01 16:08:47.408098'

UDP Characteristics Demonstrated

This program demonstrates several important UDP concepts.

Connectionless communication


UDP does not establish a connection before sending a datagram.

The client can simply send data using ``sendto()`` and specify the destination address.

Datagrams
~~~~~~~~~

UDP preserves message/datagram boundaries.

One call to ``sendto()`` sends one UDP datagram, which can be received using ``recvfrom()``.

Sender address
~~~~~~~~~~~~~~

``recvfrom()`` provides the sender's address along with the received data.

For example:

.. code-block:: text

    ('127.0.0.1', 41879)

Ephemeral ports
~~~~~~~~~~~~~~~

Because the client does not explicitly bind to a local port, the operating system assigns an ephemeral port when necessary.

The server explicitly uses port ``1027`` while the client might receive a port such as ``41879``.

Loopback networking
~~~~~~~~~~~~~~~~~~~

The program uses:

.. code-block:: text

    127.0.0.1

This is the IPv4 loopback address. Communication using this address remains on the local machine.

Example Output
--------------

Server:

.. code-block:: text

    Listening at ('127.0.0.1', 1027)
    The client at ('127.0.0.1', 41879) says b'The time is 2026-10-01 16:08:47.408098'

Client:

.. code-block:: text

    The OS assigned me the address ('0.0.0.0', 41879)
    The server ('127.0.0.1', 1027) replied 'Your data was 38 bytes long'

Important Socket Concepts
------------------------

The program provides a practical demonstration of the following concepts:

``socket()``
    Creates a socket through the operating system's networking interface.

``AF_INET``
    Selects IPv4 addressing.

``SOCK_DGRAM``
    Creates a UDP datagram socket.

``bind()``
    Associates a socket with a specific local address and port.

``sendto()``
    Sends a UDP datagram to a specified destination.

``recvfrom()``
    Receives a UDP datagram and provides the sender's address.

``getsockname()``
    Returns the socket's local address information.

``encode()``
    Converts text into bytes suitable for transmission.

``decode()``
    Converts received bytes back into text.

``argparse``
    Processes command-line arguments supplied when the program is launched.

Project Purpose
---------------

The program is intentionally small. Its purpose is not to implement a production-ready network service, but to provide a practical demonstration of how an application interacts with the operating system's socket interface.

The main concepts explored are:

.. code-block:: text

    Python application
          |
          v
      socket API
          |
          v
    Operating system
          |
          v
       UDP / IPv4
          |
          v
    Network interface
          |
          v
       destination

This provides a foundation for understanding more advanced topics such as TCP servers, concurrent clients, socket multiplexing, raw sockets, packet capture, and network protocol design.
