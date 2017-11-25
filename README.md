RSCRust Server

This is a server that implements and processes the RuneScape Classic network protocol.
It's going to implement the features that RuneScape Classic had in its version 202 mudclient release.
It's designed in the programming language Rust.  I chose this language for numerous reasons, the main reason being that I wanted to learn to use Rust effectively.  I have great interest in the memory safety of Rust.  I find it quite useful that the language doesn't require a garbage collector as a direct result of the design decisions made in the language itself, yet still gets rid of the need to manage memory manually.  The design also focusing on concurrency is nice to me.  This, as well as the portability, static linking, and efficiency of the compilers output, made this language an attractive option for me.

As far as the server implementation itself, my design goals are to make a small game engine with all content being handled by plugins that are loaded and modifiable at runtime.  The scripting language will be designed to make adding content specifically for RuneScape Classic features an easy to handle task, and will greatly simplify the process of adding new content and supporting existing content in the game.

I've put thought into how RuneScape Classic had implemented their player persistence.  I recall when I had played RuneScape Classic when that was the only version of RuneScape available to play sometime around 2002 or so probably, that occassionally upon login I'd encounter an error that referenced a login server.  I remember on occassion the friends list would be offline as well, and when messaging players I'd receive a message about the login server being down and they didn't get my message, sometimes.  This implies to me that player data was saved remotely and the world servers were independent of all saved data, and the friends list feature.  The world servers also appeared to operate fine even if the login server was offline, the consequences of it being offline simply that the friends list would stop working until it was back online, and that logging in would not work until it was back up. I also never once recall having had to deal with my player account being rolled back at all under any circumstances.  This gave me a few ideas.
I remember RSCDaemon had implemented a login server, which handled saving player data to MySQL and handled all friends list actions.  The login server RSCDaemon used was not all that great, though, and I have some better ideas for my own implementation.

I've yet to decide the actual storage medium for saved player data yet.  I'm considering a custom binary format for it to reduce storage size and to improve performance, but also taking into consideration something like SQLite and even considering MySQL or PostgreSQL a little bit, though those two are less likely to be used.
I may add support for a number of different storage mediums.

The login server will handle loading saved data from the storage medium, and saving data to the storage medium.  It will also provide an API for sending friend list private messages, adding and deleting friends from the friend list, adding and deleting player names to ignore from the ignore list.
I am most likely going to cache player data in RAM once it is loaded for the first time, speeding up subsequent logins after the initial one by reducing I/O operations made by the login server.  I suppose I will implement periodic saving of all player data in RAM, as well as an event-driven save triggered by logging out.  After being cached for so long(specific length of time not decided yet), I will remove the player data from cache to avoid the memory usage getting out of hand, though I presume player data will be inconsequentially small, for the most part.
Instead of a well-defined static list of player variables to be saved as RSCDaemon had, I will be making a dynamically growable list of variables for each player, which can be extended upon at will by the programmer.  This will make adding new data to the saves much easier than RSCDaemons approach.
To keep the design as cohesive as possible, I am considering making the connection to it on-demand rather than keeping a persistent connection to it open from the world server at all times.  If this doesn't perform well in practice, I will make the connection persistent, and if the login server goes offline for whatever reason while the world server is still in operation, I'll cache the player data modifications that are needed on the world server and make repeated attempts to reconnect until a new connection is established to the login server.  This sort of redundancy of player data being held on both servers will hopefully avoid rollbacks of player data from unexpected crashes.  The only way I see a rollback occuring would be from the login server crashing, and then the world server crashing before the login server revives, in that order.  In practice, the login server should be stable enough to where rollbacks will never be an issue.  I predict once the login server is initially implemented, there will be almost no situation where I'd need to make modifications to it or ever have to close it for any reason.

There are more things I wish to write, but for now I'm going to start implementing the game engine.

Hope you guys like the project.
