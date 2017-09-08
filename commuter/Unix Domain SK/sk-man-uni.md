# Assignment
To abstract the spec from the function description and actual codes.  
Figure out the basic var and data structure.  
The question is what is necessary and what is redundant.  
And which part can be omitted.

Another thing is how to handle err, check commuter is necessary.

# socket

### SYNOPSIS

`int socket(int domain, int type, int protocol);`

### DESCRIPTION

socket() creates an endpoint for communication and returns a file descriptor that refers to that endpoint.

- The `domain` argument specifies a communication domain; this selects the protocol family which will be used for communication. Only `AF_UNIX` for now.
- The `type` specifies the communication semantics. And temperarily:
  - SOCK_STREAM: Provides sequenced, reliable, two-way, connection-based byte streams.  An out-of-band data transmission mechanism may be supported.
- The `protocol` specifies a particular protocol to be used with the socket. set to 0;

# RETURN VALUE

On success, a file descriptor for the new socket is returned.  On error, -1 is returned, and errno is set appropriately.

# SIMPLE IMPLEMENTATION
```
    int socket(int domain, int type, int protocol){
        struct socket *sock;
        sock_create(domain, type, protocol, &sock);
        -- sock = sock_alloc();
	// sock is part of inode actually, and the detail of the inode can be omitted I guess.
	// This function simply allocate an inode, get its sock part and init them
        -- net_families[domain]->create(sock, protocol);
	// Just unix_create() here
	// Allocate a sock, bind it to the socket and init
        // Insert it to unbound sock hash_table
        
        return sock_map_fd(sock);
	// get a unused fd, create a file structure and mount it to sockfs
    }
```

# bind

### SYNOPSIS

`int bind(int sockfd, const struct sockaddr *addr, socklen_t addlen);`

### DESCRIPTION

`bind()` assigns the address specified by `addr` to the socket referred to by the file descriptor `sockfd`.  `addrlen` specifies the size, in bytes, of the address structure pointed to by addr.

### RETURN VALUE

On success, zero is returned.  On error, -1 is returned, and errno is set appropriately.

### SIMPLE IMPLEMENTATION
```
    int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen){
	sock = sockfd_lookup(sockfd);
	-- struct file *file;
        -- file = fget(fd);
        -- inode = file->f_dentry->d_inode;
        -- sock = socki_lookup(inode);
        -- check sock->file == file
        move_addr_to_kernel(addr, addrlen,address);
        unix_bind(sock, addr, addrlen);
        // check addr
	// check fs, valid dir, and file not exist
	// create a inode in that fs
	move_list(list, sk); 
    }
```

# listen

### SYNOPSIS

`int listen(int sockfd, int backlog);`

### DESCRIPTION

listen() marks the socket referred to by sockfd as a passive socket, that is, as a socket that will be used to accept incoming connection requests using accept(2).

- The sockfd argument is a file descriptor that refers to a socket of type SOCK_STREAM or SOCK_SEQPACKET.
- The backlog argument defines the maximum length to which the queue of pending connections for sockfd may grow.  If a connection request arrives when the queue is full, the client may receive an error with
an indication of ECONNREFUSED or, if the underlying protocol supports retransmission, the request may be ignored so that a later reattempt at connection succeeds.

### RETURN VALUE
On success, zero is returned.  On error, -1 is returned, and errno is set appropriately.

### SIMPLE IMPLEMENTATION
```
    int listen(int sockfd, int backlog){
	sock = sockfd_lookup(sockfd);
	check backlog;
	unix_listen(sock, backlog);
	--> check socktype;
	--> set backlog, state and pid;
    }
```

# accept

### SYNOPSIS

`int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);`

### DESCRIPTION

The accept() system call is used with connection-based socket types (SOCK_STREAM, SOCK_SEQPACKET).  It extracts the first connection request on the queue of pending connections for the listening socket, 
sockfd, creates a new connected socket, and returns a new file descriptor referring to that socket.  The newly created socket is not in the listening state.  The original socket sockfd is unaffected by this call.

- The argument `sockfd` is a socket that has been created with socket(2), bound to a local address with bind(2), and is listening for connections after a listen(2).
- The argument `addr` is a pointer to a sockaddr structure.  This structure is filled in with the address of the peer socket, as known to the communications layer. When addr is NULL, 
nothing is filled in; in this case, addrlen is not used, and should also be NULL.
- The addrlen `argument` is a value-result argument: the caller must initialize it to contain the size (in bytes) of the structure pointed to by addr; on return it will contain the actual size of the peer address.

### SIMPLE IMPLEMENTATION
```
    int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen){
	sock = sock_lookup(sockup);
	newsock = sock_alloc();
	unix_accept(sock, newsock);
	--> check sock prop
	--> skb = skb_recv_datagram(sk, 0, flags, err);

    }
```