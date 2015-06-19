This document provides the instructions needed to create and connect to a Redis database from Redis Labs.

Feel free to contribute to this document at our documents repository: https://github.com/RedisLabs/rldocs

# Creating a Redis Database #
[Redis](http://redis.io) is an open source, BSD licensed, advanced key-value cache and store. Redis Labs' products and services facilitate the deployment and management of Redis databases on and off the cloud.

## Create a Database Using the Redis Labs Enterprise Cluster ##
[Redis Labs Enterprise Cluster (RLEC)](/redis-enterprise) enables you to install an enterprise-grade cluster that acts as a container for managing and running multiple Redis databases in a highly available and scalable manner, with predictable and stable top performance.

To create your database and obtain its connection information refer to the [Redis Labs Enterprise Cluster documentation](/redis-enterprise-documentation) or click one of the links below:

 - [Installing and upgrading](/redis-enterprise-documentation/installing-and-upgrading)
 - [Initial setup - creating a new cluster](/redis-enterprise-documentation/initial-setup-creating-a-new-cluster)
 - [Database configuration / Creating a new database](/redis-enterprise-documentation/database-configuration/creating-a-new-database)

## Create a Database Using Redis Cloud ##
[Redis Cloud](/redis-cloud) is a fully-managed cloud service for hosting and running your Redis dataset in a highly-available and scalable manner, with predictable and stable top performance.

To create a database with Redis Cloud follow these steps:

 1. [Login](/#login-box) or [Sign up](/#signup-box) to Redis Cloud
 2. Using the web console, select an existing Redis Cloud subscription or add a new plan in the cloud and data region of your choosing
 3. Add a new Redis database to your subscription and configure it as appropriate
 4. Check the console for its connection information once your database becomes active

# Connecting to Redis #
To establish a connection to a Redis database, you'll need the following information:

 - The hostname or IP address of the Redis server
 - The port number that the Redis server is listening at
 - The database password (when configured with an authentication password which is **strongly recommended**)
 - The SSL certificates (when configured with SSL authentication and encryption - see [this article](/kb/read-more-ssl) for more information)

The combination of `hostname:port` is commonly referred to as the "endpoint." This information is readily obtainable from your Redis Labs Enterprise Cluster and Redis Cloud web consoles. Unless otherwise specified, our Redis databases are accessible via a single managed endpoint to ensure high availability.

You can connect to a Redis database using a wide variety of tools and libraries depending on your needs. Here's a short list:

 - Use one of the many [clients for Redis](redis.io/clients) - see below for client-specific information and examples
 - Code your own Redis client based on the [Redis Serialization Protocol (RESP)](http://redis.io/topics/protocol)
 - Make friends with Redis' own command line tool - `redis-cli` - to quickly connect and manage any Redis database (**tip:** you can also use `telnet` instead)
 - Use tools that provide a [GUI for Redis](/blog/so-youre-looking-for-the-redis-gui)

## Basic Connection Troubleshooting ##
Connecting to a remote server can be challenging. Here’s a quick checklist for common pitfalls:

 - Verify that the connection information was copy-pasted correctly <- more than 90% of connectivity issues are due to a single missing character.
 - If you're using Redis in the cloud or not inside of a LAN, consider adjusting your client's timeout settings
 - Try disabling any security measures that your database may have been set up with (e.g. Source IP/Subnet lists, Security Groups, SSL, etc...).
 - Try using a command line tool to connect to the database from your server - it is possible that your host and/port are blocked by the network.
 - If you've managed to open a connection, try sending the `INFO` command and act on its reply or error message.
 - Redis Labs Redis databases only support connecting to the default database (0( and will block some administrative commands. For more information, refer to the following:
	 - Redis Labs Enterprise Cluster: [RLEC compatability](/redis-enterprise-documentation/rlec-compatibility)
	 - Redis Cloud FAQ: [Are you fully compatible with open source Redis](/faqs#are-you-fully-compatible-with-open-source-redis)

If you encounter any difficulties or have questions please feel free to [contact our help desk](mailto:support@redislabs.com).

# Clustering Redis #
Joining multiple Redis servers into a Redis cluster is a challenging task, especially because Redis supports complex data structures and commands required by modern web applications, in high-throughput and low latency (sub-millisecond) conditions. Some of those challenges are:

 - Performing union and intersection operations over List/Set/Sorted Set
   data types across multiple shards and nodes
 - Maintaining consistency across multi-shard/multi-node architecture,
   while running (a) a SORT command over a List of Hash keys; or (b) a
   Redis transaction that includes multiple keys; or (c) a Lua script
   with multiple keys
 - Creating a simple abstraction layer that hides the complex cluster
   architecture from the user’s application, without code modifications
   and while supporting infinite scalability
 - Maintaining a reliable and consistent infrastructure in a cluster
   configuration

There are several solutions to clustering Redis, most notable of which is the [open source Redis cluster](http://redis.io/topics/cluster-spec). 

Redis Labs Enterprise Cluster and Redis Cloud were built from the ground up to provide a Redis cluster of any size while supporting all Redis commands. Your dataset is distributed across multiple shards in multiple nodes of the Redis cluster and is constantly monitored to ensure optimal performance. When needed, more shards and nodes can be added to your dataset so it can scale continuously and limitlessly.

Redis Labs clusters provide a single endpoint to connect to, and do not require any code changes or special configuration from the application’s perspective. For more information on setting up and using Redis Labs clusters, refer to the following articles:

 - Redis Labs Enterprise Cluster: [Database clustering](/redis-enterprise-documentation/database-configuration/database-clustering)
 - [Redis Cloud Cluster](/kb/redis-cloud-cluster)

# Using Redis with Ruby #
In order to use Redis with Ruby you will need a Ruby Redis client. In the following sections, we will demonstrate the use of [redis-rb](https://github.com/redis/redis-rb), a Ruby client library for Redis. Additional Ruby clients for Redis can be found under the [Ruby section](http://redis.io/clients#Ruby) of the Redis Clients page.

## Installing redis-rb ##
redis-rb's installation instructions are given in the [README file](https://github.com/redis/redis-rb). Use `gem` to install redis-rb:

    $ gem install redis

Or, include `redis-rb` in your Gemfile by adding to it the following line:

    gem 'redis'

Followed by executing `bundle install`.

## Opening a Connection to Redis Using redis-rb ##
The following code creates a connection to Redis using redis-rb:

    require 'redis'

	redis = Redis.new (
		:host => 'hostname',
	    :port => port,
	    :password => 'password')
    
To adapt this example to your code, make sure that you replace the following values with those of your database:

 - In line 4, the `:host` should be your database's hostname or IP address
 - In line 5, the `:port` should be your database's port
 - In line 6, the `:password` should be your database's password

## Using SSL and redis-rb ##
redis-rb does not support SSL connections natively. For an added security measure, you can secure the connection using [stunnel](https://redislabs.com/blog/using-stunnel-to-secure-redis) or this [redis-rb fork](https://github.com/RedisLabs/redis-rb) that has been added with SSL support.

## Reading and Writing Data with redis-rb ##
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value `bar` to the Redis key `foo`, reads it back, and prints it:

    # open a connection to Redis
    ...
 
    redis.set('foo', 'bar');
    value = redis.get('foo');
    puts value

The output of the above code should be:

	$ ruby example_redis-rb.rb
    bar

# Using Redis with Node.js #
In order to use Redis with Node.js you will need a Node.js Redis client. In following sections, we will demonstrate the use of [node_redis](https://github.com/mranney/node_redis), a complete Redis client for Node.js. Additional Node.js clients for Redis can be found under the [Node.js section](http://redis.io/clients#Node.js) of the Redis Clients page.

## Installing node_redis ##
node_redis installation instructions are given in the [README file](https://github.com/andymccurdy/redis-py/#installation). To install using `npm` issue the following command:

	$ npm install redis 
	
## Opening a Connection to Redis Using node_redis ##
The following code creates a connection to Redis using node_redis:

    var redis = require('redis');
    var client = redis.createClient(port, 'hostname', {no_ready_check: true});
    client.auth('password', function (err) {
        if (err) then throw err;
    });
    
    client.on('connect', function() {
        console.log('Connected to Redis');
    });
    
To adapt this example to your code, make sure that you replace the following values with those of your database:

 - In line 2, the first argument to `createClient` should be your database's port
 - In line 2, the second argument to `createClient` should be your database's hostname or IP address
 - In line 3, the first argument to `auth` should be your database's password

## Using SSL and node_redis ##
node_redis does not support SSL connections natively. For an added security measure, you can secure the connection using [stunnel](https://redislabs.com/blog/using-stunnel-to-secure-redis) or this [node_redis fork](https://github.com/paddybyers/node_redis) that has been added with SSL support.

## Reading and Writing Data with node_redis ##
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value `bar` to the Redis key `foo`, reads it back, and prints it:

    // open a connection to Redis
    ...
 
    client.set("foo", "bar", redis.print);
    client.get("foo", function (err, reply) {
	    if (err) then throw err;
	    console.log(reply.toString());
    });
    
The output of the above code should be:

    $ node example_node_redis.js
    Connected to Redis
    bar

# Using Redis with Python #
In order to use Redis with Python you will need a Python Redis client. In following sections, we will demonstrate the use of [redis-py](https://github.com/andymccurdy/redis-py/), a Redis Python Client. Additional Python clients for Redis can be found under the [Python section](http://redis.io/clients#Python) of the Redis Clients page.

## Installing redis-py ##
redis-py's installation instructions are given in the ["Installation" section](https://github.com/andymccurdy/redis-py/#installation) of its README file. Use `pip` to install redis-py:

    $ sudo pip install redis

You can also download the latest [redis-py release](https://github.com/andymccurdy/redis-py/releases) from the GitHub repository. To install it, extract the source and run the following command:

	$ cd redis-py
	~/redis-py$ sudo python setup.py install
	
## Opening a Connection to Redis Using redis-py ##
The following code creates a connection to Redis using redis-py:

    import redis

    r = redis.Redis(
        host='hostname',
        port=port, 
        password='password')
    
To adapt this example to your code, make sure that you replace the following values with those of your database:

 - In line 4, `host` should be set to your database's hostname or IP address
 - In line 5, `port` should be set to your database's port
 - In line 6, `password` should be set to your database's password

## Connection Pooling with redis-py ##
redis-py provides a connection pooling mechanism as explained in the [Connection Pools section](https://github.com/andymccurdy/redis-py#connection-pools) of its README file. Since connection pooling is enabled by default, no special actions are required to use it.

## Using SSL and redis-py ##
redis-py is the second Redis client that natively supported SSL. Use the `SSLConnection` class or simply instantiate your connection pool using a `rediss://`-URL and the `from_url` method, like so:

    r = redis.Redis( url='rediss://:password@hostname:port/0',
        password='password',
        ssl_keyfile='path_to_keyfile',
        ssl_certfile='path_to_certfile',
        ssl_cert_reqs='required',
        ssl_ca_certs='path_to_ca_cert')

## Reading and Writing Data with redis-py ##
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value `bar` to the Redis key `foo`, reads it back, and prints it:

    # open a connection to Redis
    ...
 
    r.set('foo', 'bar')
    value = r.get('foo')
    print(value)

The output of the above code should be:

    $ python example_redis-py.py
    bar

# Using Redis with Java #
In order to use Redis with Java you will need a Java Redis client. In following sections, we will demonstrate the use of [lettuce] (https://github.com/mp911de/lettuce/) and [Jedis](https://github.com/xetorthio/jedis). Additional Java clients for Redis can be found under the [Java section](http://redis.io/clients#Java) of the Redis Clients page.

## installing lettuce ##
 
lettuce' installation instructions are given in the "How do I Use it?" section of its README file. Use lettuce by declaring the following Maven dependency:
 
    <dependency>
        <groupId>biz.paluch.redis</groupId>
        <artifactId>lettuce</artifactId>
        <version>3.2.Final</version>
    </dependency>
 
You can also download the latest lettuce release from the GitHub repository: https://github.com/mp911de/lettuce/wiki/Download
 
## opening a connection to Redis using lettuce ##
 
The following code creates a connection to Redis using lettuce:
 
    import com.lambdaworks.redis.*;
 
    public class ConnectToRedis {
 
        public static void main(String[] args) {
            // Syntax: redis://[password@]host[:port][/databaseNumber]
            RedisClient redisClient = new RedisClient(RedisURI.create("redis://password@localhost:6379/0"));
            RedisConnection<String, String> connection = redisClient.connect();
 
            System.out.println("Connected to Redis");
 
            connection.close();
            redisClient.shutdown();
        }
    }

To adapt this example to your code, make sure that you replace the following values with those of your database:
 
 - In line 7, the URI contains the password. The argument should be your database's password. Remove `password@` to connect without authentication
 - In line 7, the URI contains the port. The argument should be your database's port.
 - In line 7, the URI contains the database number. The argument should be your database number for the initial connection.

 hostname, port and the database number. 
     Change these values according to your hostname, port and database number.
     If you can remove the password, port number and database number to use default values (no authentication, port 6379 and database number 0)
 
lettuce is thread-safe, and the same lettuce connection can be used from different threads. Using multiple connections is also possible.
 
If you're using Spring, use the following plain Spring XML to create a lettuce instance:
 
    <bean id="RedisClient" class="com.lambdaworks.redis.support.RedisClientFactoryBean">
        <property name="uri" value="redis://localhost:6379"/>
    </bean>
 
 
and use it then within your managed beans
 
    import com.lambdaworks.redis.*;
    import org.springframework.beans.factory.annotation.Autowired;
 
    public class MySpringBean {
 
        private RedisClient redisClient;
 
        @Autowired
        public void setRedisClient(RedisClient redisClient) {
            this.redisClient = redisClient;
        }
 
        public String ping() {
 
            RedisConnection<String, String> connection = redisClient.connect();
            String result = connection.ping();
            connection.close();
            return result;
        }
    }
 
Once your standalone-application exits, remember to shutdown lettuce by using the shutdown method:
 
    client.shutdown();
 
If you are using Spring and CDI, the frameworks will manage the resources for you and you do not have to close the client using the shutdown method.
 
## using ssl and lettuce ##
 
For an added security measure, you can secure the connection using SSL connections. lettuce supports SSL connections natively.
 
    import com.lambdaworks.redis.*;
 
    public class ConnectToRedisSSL {
 
        public static void main(String[] args) {
            // Syntax: rediss://[password@]host[:port][/databaseNumber]
            RedisClient redisClient = new RedisClient(RedisURI.create("rediss://password@localhost:6443/0"));
            RedisConnection<String, String> connection = redisClient.connect();
            System.out.println("Connected to Redis using SSL");
 
            connection.close();
            redisClient.shutdown();
        }
    }
 
## reading and writing data with lettuce ##
 
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value bar to the Redis key foo, reads it back, and prints it:
 
    // open a connection to Redis
    ...
 
    connection.set("foo", "bar");
    String value = connection.get("foo");
    System.out.println(value);
 
The output of the above code should be:
 
    $ java ReadWriteExample
    Connected to Redis
    bar
 
 
## connecting to Redis using Redis Sentinel ##
 
Redis Sentinel is an HA-Registry to monitor your Redis instances. You connect to one or more Redis Sentinel instances to obtain the connection address of the current Redis master. You need to specify the `redis-sentinel` scheme and one or more Redis Sentinel addresses within the connection URI to connect to Redis using Redis Sentinel:
 
    import com.lambdaworks.redis.*;
 
    public class ConnectToRedisUsingRedisSentinel {
 
        public static void main(String[] args) {
            // Syntax: redis-sentinel://[password@]host[:port][,host2[:port2]][/databaseNumber]#sentinelMasterId
            RedisClient redisClient = new RedisClient(RedisURI.create("redis-sentinel://localhost:26379,localhost:26380/0#mymaster"));
            RedisConnection<String, String> connection = redisClient.connect();
 
            System.out.println("Connected to Redis using Redis Sentinel");
 
            connection.close();
            redisClient.shutdown();
        }
    }
 
## connecting to a Redis Cluster ##
 
You can connect to a Redis Cluster using lettuce's RedisClusterClient. The client has a view on the cluster topology and routes calls for keys on particular slot-hashes to the appropriate node transparently. Once you are connected use the client as you would for standalone operations. Keep in mind, that multi-key operations like `zunionstore` have to operate on one slot-hash.
 
    import com.lambdaworks.redis.*;
    import com.lambdaworks.redis.cluster.*;
 
    public class ConnectToRedisCluster {
 
        public static void main(String[] args) {
            // Syntax: redis-cluster://[password@]host[:port][/databaseNumber]
            RedisClusterClient redisClient = new RedisClusterClient(RedisURI.create("redis://password@localhost:7379"));
            RedisAdvancedClusterConnection<String, String> connection = redisClient.connectCluster();
 
            System.out.println("Connected to Redis");
 
            connection.close();
            redisClient.shutdown();
        }
    }

## Installing Jedis##
Jedis' installation instructions are given in the ["How do I Use it?" section](https://github.com/xetorthio/jedis/#how-do-i-use-it) of its README file. Use Jedis by declaring the following Maven dependency:

    <dependency>
	    <groupId>redis.clients</groupId>
	    <artifactId>jedis</artifactId>
	    <version>2.6.2</version>
	    <type>jar</type>
	    <scope>compile</scope>
	</dependency>

You can also download the latest [Jedis release](https://github.com/xetorthio/jedis/releases) from the GitHub repository. To build it, extract the source and run the following command:

	$ cd jedis
	~/jedis$ make package

## Opening a Connection to Redis Using Jedis ##
The following code creates a connection to Redis using Jedis:

	import redis.clients.jedis.Jedis;
	 
	public class JedisExample {
	 
	  public static void main(String[] args) throws Exception {
	    Jedis jedis = new Jedis("hostname", port);
	    jedis.auth("password")
	    System.out.println("Connected to Redis");
	  }
	}

To adapt this example to your code, make sure that you replace the following values with those of your database:

 - In line 6, the first argument to `Jedis` should be your database's hostname or IP address
 - In line 6, the second argument to `Jedis` should be your database's port
 - In line 7, the argument to `auth` should be your database's password

## Connection Pooling with Jedis ##
Jedis isn't thread-safe, and the same Jedis instance shouldn't be used from different threads. To overcome the overhead of multiple Jedis instances and connection maintenance, use `JedisPool`. To use JedisPool you'll have to have `Apache Commons Pool 2.3` available - download it  [here]( http://commons.apache.org/proper/commons-pool/download_pool.cgi) or add the following Maven dependency:

	<dependency>
	    <groupId>org.apache.commons</groupId>
	    <artifactId>commons-pool2</artifactId>
	    <version>2.3</version>
	</dependency>

The following code instantiates a pool of connections:


    JedisPool pool = new JedisPool(new JedisPoolConfig(), "hostname", port, Protocol.DEFAULT_TIMEOUT, "password");

If you're using Spring, use the following Plain Spring XML to create JedisPool:

	<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
	        <constructor-arg index="0" ref="jedisPoolConfig" />
	        <constructor-arg index="1" value="hostname" />
	        <constructor-arg index="2" value="port" />
	        <constructor-arg index="3" value="Protocol.DEFAULT_TIMEOUT" />
	        <constructor-arg index="4" value="password" />
	    </bean>
	
	    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig" >
	</bean>

JedisPool is thread-safe and can be stored in a static variable and shared among threads. The following code gets a Jedis instance from the JedisPool:

	Jedis redis = null;
	    try
	    {
	        redis = redisPool.getResource();
	        return redis.get(keyName);
	    }
	    catch (JedisConnectionException e)
	    {
	        if (redis != null)
	        {
	            redisPool.returnBrokenResource(redis);
	            redis = null;
	        }
	        throw e;
	    }
	    finally
	    {
	        if (redis != null)
	        {
	            redisPool.returnResource(redis);
	        }
	    }
	}

Once your application exits, remember to dispose of the `JedisPool` by using the `destroy` method:

    pool.destroy();

## Using SSL and Jedis ##
Jedis does not support SSL connections natively. For an added security measure, you can secure the connection using [stunnel](https://redislabs.com/blog/using-stunnel-to-secure-redis) or this [Jedis fork](https://github.com/RedisLabs/jedis) that has been added with SSL support.

## Reading and Writing Data with Jedis ##
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value `bar` to the Redis key `foo`, reads it back, and prints it:

    // open a connection to Redis
    ...
 
    jedis.set("foo", "bar");
    String value = jedis.get("foo");
    System.out.println(value);

The output of the above code should be:

    $ java JedisExample
    Connected to Redis
    bar

# Using Redis with C #
In order to use Redis with C you will need a C Redis client. In following sections, we will demonstrate the use of [hiredis](https://github.com/redis/hiredis), a minimalistic C client for Redis. Additional C clients for Redis can be found under the [C section](http://redis.io/clients#C) of the Redis Clients page.

## Installing hiredis ##
Download the latest [hiredis release](https://github.com/redis/hiredis/releases) from the GitHub repository.

## Opening a Connection to Redis Using hiredis ##
The following code creates a connection to Redis using  hiredis' synchronous API:

	#include "hiredis.h"

	redisContext *c = redisConnect("hostname", port);
	if (c != NULL && c->err) {
	    printf("Error: %s\n", c->errstr);
	    // handle error
	} else {
		printf("Connected to Redis\n");
	}

	redisReply *reply;
	reply = redisCommand(c, "AUTH password");
	freeReplyObject(reply);

	...

	redisFree(c);
    
To adapt this example to your code, make sure that you replace the following values with those of your database:

 - In line 1, the first argument to `redisConnect` should be your database's hostname or IP address
 - In line 1, the second argument to `redisConnect` should be your database's port
 - In line 6, replace "password" with your database's password

## Using SSL and hiredis ##
hiredis does not support SSL connections natively. For an added security measure, you can secure the connection using [stunnel](https://redislabs.com/blog/using-stunnel-to-secure-redis).

## Reading and Writing Data with hiredis ##
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value `bar` to the Redis key `foo`, reads it back, and prints it:

    // open a connection to Redis
    ...
 
	redisReply *reply;

	reply = redisCommand(c,"SET %s %s","foo","bar");
	freeReplyObject(reply);

	reply = redisCommand(c,"GET %s","foo");
	printf("%s\n",reply->str);
	freeReplyObject(reply);
    
The output of the above code should be:

    $ gcc example_hiredis.c -o example_hiredis
    $ ./example_hiredis
    Connected to Redis
    bar

# Using Redis with .Net C# #
In order to use Redis with C# you will need a C# Redis client. In following sections, we will demonstrate the use of [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis), General purpose Redis client. Additional C# clients for Redis can be found under the [C# section](http://redis.io/clients#C#) of the Redis Clients page.

## Installing StackExchange.Redis ##
StackExchange.Redis' installation instructions are given in the ["Installation" section](https://github.com/StackExchange/StackExchange.Redis#Installation) of its README file. It can be installed via the nuget package manager console with the following command:

    PM> Install-Package StackExchange.Redis
    
## Opening a Connection to Redis Using StackExchange.Redis ##
The following code creates a connection to Redis using  StackExchange.Redis:

	using StackExchange.Redis;
	
	readonly ConnectionMultiplexer muxer = ConnectionMultiplexer.Connect("hostname:port,password=password");
	IDatabase conn = muxer.GetDatabase();
    
To adapt this example to your code, make sure that you replace the following values with those of your database:

 - In line 1, the first part of the string argument to `Connect` should be your database's endpoint
 - In line 1, the second part of the string argument to `Connect` should be your database's password

## Connection Pooling with StackExchange.Redis ##
While StackExchange.Redis does not provide direct means for conventional connection pooling, we recommend you **share and reuse** the ConnectionMultiplexer object. The ConnectionMultiplexer object should not be created per operation - it is to be created only once at the beginning and reused for the duration of the run. ConnectionMultiplexer is thread-safe so it can be safely shared between threads. For more information, refer to StackExchange.Redis’ [Basic Usage document](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md).

## Using SSL and StackExchange.Redis ##
StackExchange.Redis is the first Redis client that natively supported SSL. The following code opens an SSL connection:

	using StackExchange.Redis;
	using System.Security.Cryptography.X509Certificates;
	using System.Net.Security;

	var options = new ConfigurationOptions
    {
	    EndPoints = { "hostname:port" },
	    Password = "password",
	    Ssl = true
	};
	options.CertificateSelection += delegate {
                return new X509Certificate2("d:\path\filname.pfx", "");
            };

	readonly ConnectionMultiplexer muxer = ConnectionMultiplexer.Connect(options);
	IDatabase conn = muxer.GetDatabase();
	
 - In line 6, the first part of the string argument should be your database's endpoint or IP address
 - In line 6, the second part of the string argument should be your database's port
 - In line 7, the string argument should be your database's password
 - In line 11, replace with the path to your .pfx file

### Converting Certificates from .key to .pfx Format ###
To easily convert a .key certificate to .pfx format use OpenSSL:

	$ openssl pkcs12 -export -in user.crt -inkey user_private.key -out certificate.pfx 

**Important:** if you're using a self-signed certificate, remember to install it on your server with the Certificate Manager tool.

### Using SSL and a StackExchange.Redis-based Provider ###
Sometimes you need to use a 3rd-party library, such as when running a session on a cache provider that connects to Redis with the StackExchange.Redis client. When you need to provide an SSL certificate for the connection and the 3rd-party library does not expose a public interface for it, you can "sideload" the certificate to StackExchange.Redis by setting the following environment variables:

- `SERedis_ClientCertPfxPath should` be set to the path of your .pfx file
- `SERedis_ClientCertPassword` should be set to the password of your .pfx file

## Reading and Writing Data with StackExchange.Redis ##
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value `bar` to the Redis key `foo`, reads it back, and prints it:

    ///<summary>
    ///open a connection to Redis
	///</summary>
	...

	conn.StringSet("foo", "bar");
	var value = conn.StringGet("foo");
	Console.WriteLine(value);
    
The output of the above code should be:

    bar

# Using Redis with PHP #
In order to use Redis with PHP you will need a PHP Redis client. In following sections, we will demonstrate the use of [Predis](https://github.com/nrk/predis), a flexible and feature-complete Redis client library for PHP >= 5.3. Additional PHP clients for Redis can be found under the [PHP section](http://redis.io/clients#PHP) of the Redis Clients page.

## Installing Predis ##
Predis' installation instructions are given in the ["How to Use Predis" section](https://github.com/nrk/predis#how-to-use-predis) of its README file. The recommended method for installing Predis is to use Composer and install it from [Packagist](https://packagist.org/packages/predis/predis) or the dedicated [PEAR channel](http://pear.nrk.io/).

You can also download the latest [Predis release](https://github.com/nrk/predis/releases) from the GitHub repository.

### Opening a Connection to Redis Using Predis ###
The following code creates a connection to Redis using Predis:

	<?php
	require "predis/autoload.php";
	Predis\Autoloader::register();
	
    $redis = new Predis\Client(array(
        "scheme" => "tcp",
        "host" => "hostname",
        "port" => port,
        "password” => “password"));
    echo "Connected to Redis";
	?>

Unless you've installed Predis with Composer, you'll need to include the 2nd and 3rd lines of code to load and register the Predis client library. To adapt this example to your code, make sure that you replace the following values with those of your database:

 - In line 8, `host` should refer to your database's hostname or IP address
 - In line 9, `port` should be your database's port
 - In line 10, `password` should be your database's password

## Persistent Connections with Predis
Predis supports the use ofing persistent connections, which are recommended practice to minimizeconnection management overhead. To enable persistent connections, use the `persistent` connection attribute as shown in the following snippet:

    $redis = new Predis\Client(array(
        "scheme" => "tcp",
        "host" => "hostname",
        "port" => port,
        "password” => "password",
        "persistent" => "1"));

## Using SSL and Predis ##
does not support SSL connections natively. For an added security measure, you can secure the connection using [stunnel](https://redislabs.com/blog/using-stunnel-to-secure-redis) or this [Predis fork](https://github.com/RedisLabs/predis) that has been added with SSL support.

## Reading and Writing Data with Predis ##
Once connected to Redis, you can start reading and writing data. The following code snippet writes the value `bar` to the Redis key `foo`, reads it back, and prints it:

    // open a connection to Redis
    ...
 
    $redis->set("foo", "bar");
    $value = $redis->get("foo");
    var_dump($value);

The output of the above code should be:

    $ php predis_example.php
	Connected to Redis
	string(3) "bar"

# Using Redis with Drupal 7#

## Installing Redis for Drupal ##
Follow these steps to install Redis as a cache for Drupal:
 1. [Install Predis](#installing-predis) under `sites/all/libraries/predis`
 2.  Download and install the [Drupal Redis module](https://drupal.org/project/redis)

## Configuring Redis for Drupal ##
To configure Drupal to use Redis as a cache, append the following lines to your `settings.php` file:

    $conf['redis_client_interface'] = 'Predis';
    $conf['redis_client_host'] = 'hostname';
    $conf['redis_client_port'] = port;
    $conf['redis_client_password'] = 'password';
    $conf['lock_inc'] = 'sites/all/modules/contrib/redis/redis.lock.inc';
    $conf['cache_backends'][] = 'sites/all/modules/contrib/redis/redis.autoload.inc';
    $conf['cache_default_class'] = 'Redis_Cache';

 - In line 2, replace `hostname` with your database's hostname or IP address
 - In line 3, replace `port` with your database's port
 - In line 4, replace `password` with your database's password
