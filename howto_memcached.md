This document provides the instructions needed to create and connect to a Memcached Cloud bucket from Redis Labs.

Feel free to contribute to this document at our documents repository: https://github.com/RedisLabs/rldocs

# Creating a Memcached Bucket #
Memcached is a free & open source, high-performance, distributed memory object caching system, generic in nature, but intended for use in speeding up dynamic web applications by alleviating database load. Redis Labs' products and services facilitate the deployment and management of Memcached buckets on and off the cloud.

## Create a Bucket Using the Redis Labs Enterprise Cluster ##
[Redis Labs Enterprise Cluster (RLEC)](/redis-enterprise) enables you to install an enterprise-grade cluster that acts as a container for managing and running multiple Memcached buckets in a highly available and scalable manner, with predictable and stable top performance.

To create your bucket and obtain its connection information refer to the [Redis Labs Enterprise Cluster documentation](/redis-enterprise-documentation) or click one of the links below:

 - [Installing and upgrading](/redis-enterprise-documentation/installing-and-upgrading)
 - [Initial setup - creating a new cluster](/redis-enterprise-documentation/initial-setup-creating-a-new-cluster)
 - [Database configuration / Creating a new database](/redis-enterprise-documentation/database-configuration/creating-a-new-database)

## Create a Bucket Using Memcached Cloud ##
[Memcached Cloud](/memcached-cloud) is a fully-managed, reliable service for hosting and running your Memcached.

To create a bucket with Memcached Cloud follow these steps:

 - [Login](/#login-box) or [Sign up](/#signup-box) to Memcached Cloud
 - Using the web console, select an existing Memcached Cloud subscription or add a new plan in the cloud and data region of your choosing
 - Add a new Memcached bucket to your subscription and configure it
 - As your bucket becomes active, the console will display its connection information

**Important note:** Memcached buckets from Redis Labs are powered by Redis and as such offer several advanced capabilities that are not included with the standard memcached server. While the use of these capabilities is strictly optional, consider the following advantages:

 - Depending on your chosen plan, your bucket can be configured with data persistency for durability purposes so you don’t need to synchronize your cache’s contents to another SQL or NoSQL database
 - You can import and export data to and from your bucket
 - You can use `KEYS` and `MONITOR`, just like in Redis (for more information read [this blog post](/blog/finally-you-can-see-whats-stored-in-your-memcached)

# Connecting to Memcached #
To establish a connection to a Memcached bucket, you'll need the following information:

 - The hostname or IP address of the Memcached server
 - The port number that the Memcached server is listening at
 - The bucket's username and password (when configured with SASL authentication which is **strongly recommended**)

The combination of the `hostname:port` is commonly referred to as the "endpoint." This information is readily obtainable from your web console. Memcached buckets from Redis Labs are accessible via a single managed endpoint to ensure high availability

You can connect to a Memcached bucket using a wide variety of tools and libraries depending on your needs. Here's a short list:

 - Use one of the many [clients for Memcached](https://code.google.com/p/memcached/wiki/Clients) (see below for client-specific information and examples)
 - Code your own Memcached client, based on the [Memcached protocols](https://code.google.com/p/memcached/wiki/NewProtocols) 
 - Make friends with Redis Labs' own command line tool - [`bmemcached-cli`](https://github.com/RedisLabs/bmemcached-cli) - to quickly connect and manage any Memcached bucket (**tip:** you can also use `telnet` instead)

## Basic Connection Troubleshooting ##
Connecting to a remote server can be challenging. Here’s a quick checklist of common pitfalls:

 - Verify that the connection information was copy-pasted correctly - more than 90% of connectivity issues are due to a single missing character
 -  If you're using Memcached in the cloud or not inside of a LAN, consider adjusting your client's timeout settings
 - If you’re using SASL authentication, note that the default Memcached text protocol can’t be used with SASL authentication - you’ll only be able to use tools and clients that use Memcached’s binary protocol
 - Try disabling any security measures that your database may have been set up with (e.g. Source IP/Subnet lists, Security Groups, SASL, etc.)
 - Try using a command line tool to connect to the database from your server - it is possible that your host and/port are blocked by the network.
 -  If you've managed to open a connection, try sending the `STATS` command and take note of its reply or error message

If you encounter any difficulties or have questions please feel free to [contact our help desk](mailto:support@redislabs.com).

# Using Memcached with Ruby #
In order to use Memcached with Ruby you will need a Ruby Memcached client. In the following sections, we will demonstrate the use of [dalli](https://github.com/mperham/dalli), a high-performance memcached client for Ruby.

## Installing Dalli ##
Dalli's installation instructions are given in the ["Installation and usage" section](https://github.com/mperham/dalli#installation-and-usage) of its README file. After installing `memcached`, use `gem` to install Dalli:

    foo@bar:~$ gem install dalli

Or, include `dalli` in your Gemfile by adding the following line:

    gem 'dalli'

Followed by executing `bundle install`.

## Opening a Connection to Memcached Using Dalli ##
The following code creates a connection to Memcached using Dalli:

    require 'dalli'

	options = { :username => 'username', :password => 'password' }
	mc = Dalli::Client.new('hostname:port', options)
    
To adapt this example to your code, make sure that you replace the following values with those of your bucket:

 - In line 3, `:username` should be set with your bucket's username
 - In line 3, `:password` should be set with your bucket's password
 - In line 4, the first argument should be your bucket's endpoint

## Reading and Writing Data with Dalli ##
Once connected to Memcached, you can start reading and writing data. The following code snippet writes the value `bar` to the Memcached key `foo`, reads it back, and prints it:

    # open a connection to Memcached
    ...
 
	mc.set('foo', 'bar')
	value = dc.get('foo')
    puts value

The output of the above code should be:

    $ ruby example_dalli.rb
    bar

## Using Dalli with Rails 3 and 4 ##
To use Memcached as a cache store for Rails, modify the file `config/environments/production.rb` with the following:

	config.cache_store = :dalli_store,
                    ('hostname:port',
                    {:username => 'username',
                     :password => 'password'
                    }

Once configured, you can use the following snippet to use the cache in your Rails app:
	
	Rails.cache.write("foo", "bar")
	value = Rails.cache.read("foo")
	
# Using Memcached with Node.js #
In order to use Memcached with Node.js you will need a Node.js Memcached client. In the following sections, we will demonstrate the use of [MemJS](https://github.com/alevy/memjs), a Memcache client for Node.js using the binary protocol and SASL authentication.

## Installing MemJS ##
MemJS' installation instructions are given in the ["Installation" section](https://github.com/alevy/memjs#installation) of its README file. To install, use `npm` as follows:

    $ npm install memjs
    
## Opening a Connection to Memcached Using MemJS ##
The following code creates a connection to Memcached using MemJS:

	var memjs = require('memjs');
	
	var mc = memjs.Client.create('hostname:port', {
	  username: 'username',
	  password: 'password'
	});    

To adapt this example to your code, make sure that you replace the following values with those of your bucket:

 - In line 3, the first argument should be your bucket's endpoint
 - In line 4, `username` should be set with your bucket's username
 - In line 5, `password` should be set with your bucket's password

## Reading and Writing Data with MemJS ##
Once connected to Memcached, you can start reading and writing data. The following code snippet writes the value `bar` to the Memcached key `foo`, reads it back, and prints it:

    // open a connection to Memcached
    ...
 
	mc.set('foo', 'bar');
	mc.get('foo', function (err, value, key) {
		if (value != null) {
		    console.log(value.toString());
		}
	});
The output of the above code should be:

    $ node example_memjs.js
    bar
	
# Using Memcached with Python #
In order to use Memcached with Python you will need a Python Memcached client. In the following sections, we will demonstrate the use of [bmemcached](https://github.com/jaysonsantos/python-binary-memcached), a pure Python module (thread-safe) to access Memcached via its binary protocol with SASL auth support.

## Installing bmemcached ##
bememcached's installation instructions are given in the ["Installing" section](https://github.com/jaysonsantos/python-binary-memcached/#installing) of its README file. Use `pip` to install bmemcached:

    $ sudo pip install python-binary-memcached

## Opening a Connection to Memcached Using bmemcached ##
The following code creates a connection to Memcached using MemJS:

	import bmemcached

	mc = bmemcached.Client('hostname:port', 'username', 'password')
	
To adapt this example to your code, make sure that you replace the following values with those of your bucket:

 - In line 3, the first argument should be your bucket's endpoint
 - In line 3, the second argument should be set with your bucket's username
 - In line 3, the third argument should be set with your bucket's password

## Reading and Writing Data with bmemcached ##
Once connected to Memcached, you can start reading and writing data. The following code snippet writes the value `bar` to the Memcached key `foo`, reads it back, and prints it:

    # open a connection to Memcached
    ...
 
    mc.set('foo', 'bar')
    value = r.get('foo')
    print(value)

The output of the above code should be:

    $ python example_bmemcached.py
    bar
	
# Using Memcached with Java #
In order to use Memcached with Java you will need a Java Memcached client. In the following sections, we will demonstrate the use of [spymemcached](https://github.com/dustin/java-memcached-client), a simple, asynchronous, single-threaded Memcached client written in Java.

## Installing spymemcached ##
To use spymemcached, begin by  adding the following repository to `Maven`:

	<repositories>
	    <repository>
	      <id>spy</id>
	      <name>Spy Repository</name>
	      <layout>default</layout>
	      <url>http://files.couchbase.com/maven2/</url>
	      <snapshots>
	        <enabled>false</enabled>
	      </snapshots>
	    </repository>
	</repositories>

Next, declare the following Maven dependency:
	
	<dependency>
	  <groupId>spy</groupId>
	  <artifactId>spymemcached</artifactId>
	  <version>2.11.6</version>
	  <scope>provided</scope>
	</dependency>
	
Even if you're not using Maven, you can download the latest [spymemcached .jar files](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22net.spy%22%20AND%20a%3A%22spymemcached%22) directly from The Maven Central Repository.

## Opening a Connection to Memcached Using spymemcached ##
The following code creates a connection to Memcached using  spymemcached:

	import java.io.IOException;
	import net.spy.memcached.*;
	
	public class spymemcachedExample {
		public static void main(String[] args) {
		    AuthDescriptor ad = new AuthDescriptor("PLAIN", 
		        new PlainCallbackHandler("username", "password"));
	
			try {
				MemcachedClient mc = new MemcachedClient(
				new ConnectionFactoryBuilder()
					.setProtocol(ConnectionFactoryBuilder.Protocol.BINARY)
					.setAuthDescriptor(ad).build(),
				AddrUtil.getAddresses(new String[] { "hostname:port" }));
			} catch (IOException e) {
				// handle exception
			}
		}
	}
		
To adapt this example to your code, make sure that you replace the following values with those of your bucket:

 - In line 7, replace `username` with your bucket's username
 - In line 7, replace `password` with your bucket's password
 - In line 14, the argument should be your bucket's endpoint

## Reading and Writing Data with spymemcached ##
Once connected to Memcached, you can start reading and writing data. The following code snippet writes the value `bar` to the Memcached key `foo`, reads it back, and prints it:

    // open a connection to Memcached
    ...
 
	mc.set("foo", 0, "bar");
	String value = mc.get("foo");
	System.out.println(value);

The output of the above code should be:

    $ java spymemcachedExample
    bar
	
# Using Memcached with PHP #
In order to use Memcached with PHP you will need a PHP Memcached client. In the following sections, we will demonstrate the use of [PECL memcached](http://php.net/manual/en/book.memcached.php), an extension that uses the libmemcached library to provide an API for communicating with Memcached servers as well as a session handler (memcached).

## Installing PECL memcached ##
PECL memcached's installation instructions are given in its ["Installation" page](http://php.net/manual/en/memcached.installation.php). To install it, use the following command:

	pecl install memcached

**Note:** PECL memcached depends on having [libmemcached](http://libmemcached.org/libMemcached.html) installed as per the [PECL memcached requirements page](http://php.net/manual/en/memcached.requirements.php).

Once installed, you should review the `/etc/php5/mods-available/memcached.ini` file or your site's `php.ini` file and verify the following settings in it:

1. SASL authentication (`memcached.use_sasl`) - to use a SASL-enabled Memcached  Cloud bucket, enable this setting by setting it to *1*
2. Session redundancy (`memcache.redundancy`) - Memcached Cloud features built-in redundancy so using the one provided by the extension isn't required. Disable this setting by setting it to *0*
2. Hash strategy (`memcache.hash_strategy`) - as there is only one hostname per bucket, Memcached Cloud is agnostic to the hash strategy you use, i.e. *consistent* (the default) or *standard* can be used
3. Hash Function (`memcache.hash_function`) - Memcached Cloud is agnostic to the hash function used by the PECL memcached, so either *crc32* (the default) or *fnv* can be used

**Note:** if you've installed PECL memcached, you should consider restarting your web server to apply the changes.

## Opening a Connection to Memcached Using PECL memcached ##
The following code creates a connection to Memcached using PECL memcached:

	<?php
		$mc = new Memcached();
		$mc->setOption(Memcached::OPT_BINARY_PROTOCOL, true);
		$mc->addServer('hostname.redislabs.com', 11211);
		$mc->setSaslAuthData('username', 'password');
	?>
	
To adapt this example to your code, make sure that you replace the following values with those of your bucket:

 - In line 4, the first argument to `addServer` should be your bucket's hostname or IP address
 - In line 4, the second argument to `addServer` should be your bucket's port
 - In line 5, the first argument to `setSaslAuthData` should be your bucket's username
 - In line 5, the second argument to `setSaslAuthData` should be your bucket's password

## Reading and Writing Data with PECL memcached ##
Once connected to Memcached, you can start reading and writing data. The following code snippet writes the value `bar` to the Memcached key `foo`, reads it back, and prints it:

    // open a connection to Memcached
    ...
 
	$mc->set('foo', 'bar');
	value = $mc->get('foo');
	echo value;

The output of the above code should be:

    $ php memcached.php
    bar
	
# Using Memcached with .NET C# #
In order to use Memcached with .NET C# you will need a .NET C# Memcached client. In the following sections, we will demonstrate the use of [EnyimMemcached](https://github.com/enyim/EnyimMemcached), a .NET client library for Memcached written in C#.

## Installing EnyimMemcached ##
To install EnyimMemcached, run the following command in the Package Manager Console:

	PM> Install-Package EnyimMemcached
## Opening a Connection to Memcached Using EnyimMemcached ##
The following code creates a connection to Memcached using  EnyimMemcached:

	using Enyim.Caching;
	using Enyim.Caching.Configuration;
	using Enyim.Caching.Memcached;

	MemcachedClientConfiguration config = new MemcachedClientConfiguration();
	config.Servers.Add(new IPEndPoint("hostname", port));
	config.Protocol = MemcachedProtocol.Binary;
	config.Authentication.Type = typeof(PlainTextAuthenticator);
	config.Authentication.Parameters["userName"] = "username";
	config.Authentication.Parameters["password"] = "password";
	config.Authentication.Parameters["zone"] = "";

	var mc = new MemcachedClient(config);

To adapt this example to your code, make sure that you replace the following values with those of your bucket:

 - In line 6, the first argument to `IPEndPoint` should be your bucket's hostname or IP address
 - In line 6, the second argument to `IPEndPoint` should be your bucket's port
 - In line 9, `userName` should be set with your bucket's username
 - In line 10, `password` should be set with your bucket's password

## Reading and Writing Data with EnyimMemcached ##
Once connected to Memcached, you can start reading and writing data. The following code snippet writes the value `bar` to the Memcached key `foo`, reads it back, and prints it:


    ///<summary>
    ///open a connection to Memcached
	///</summary>   
    ...
 
	mc.Store(StoreMode.Set, "foo", "bar");
	var value = mc.Get("foo");
	Console.WriteLine(value);
	
The output of the above code should be:

    bar
	
# Using Memcached with WordPress #
To use Memcached as a cache for your WordPress installation, follow these steps as described in the documentation of the [Memcached Cloud plugin for WordPress](https://wordpress.org/plugins/memcached-cloud/):

1. Make sure that your Drupal server is installed with the [libmemcached](http://libmemcached.org/libMemcached.html) and the [PECL memcached](#installing-pecl-memcached) extension.
2. Copy the file `object-cache.php` from the [Memcached Cloud plugin for WordPress .ZIP file](https://downloads.wordpress.org/plugin/memcached-cloud.zip) to `wp-content/object-cache.php` of your WordPress installation.
3. Add the following definitions to your WordPress' installation `wp-config.php` file:

		global $memcached_servers;
		$memcached_servers = array( array( 'hostname', port ) );

		global $memcached_username;
		$memcached_username = 'username';

		global $memcached_password;
		$memcached_password = 'password';

 - In line 3, the first argument should be your bucket's hostname or IP address
 - In line 3, the second argument should be your bucket's port
 - In line 5, the value should be your bucket's username
 - In line 7, the value should be your bucket's password
 
 # Using Memcached with Django #
To use Memcached as a cache for your Django website
edit your Django's `settings.py` file as described in the ["Django's cache framework"](https://docs.djangoproject.com/en/1.7/topics/cache/) page of Django's documentation. Add the following `CACHE` settings to `settings.py`:

	os.environ['MEMCACHE_SERVERS'] = 'hostname:port'	os.environ['MEMCACHE_USERNAME'] = 'username'
	os.environ['MEMCACHE_PASSWORD'] = 'password'
	
	CACHES = {
	    'default': {
	        'BACKEND': 'django.core.cache.backends.memcached.PyLibMCCache',
	        'BINARY': True
	    }
	}

 - In line 1, the value should be your bucket's endpoint
 - In line 2, the value should be your bucket's username
 - In line 3, the value should be your bucket's password

Note: make sure you have [`django-pylibmc`](https://github.com/jbalogh/django-pylibmc) installed on your Django server by running the following command:

    $ sudo pip install django-pylibmc
	
# Using Memcached with Drupal #
To use Memcached as a cache for your Drupal site, follow the instructions provided in ["Memcached / Installation"](https://www.drupal.org/node/1131468) page of  Drupal's documentation. The required steps are:

 1. Make sure that your Drupal server is installed with [libmemcached](http://libmemcached.org/libMemcached.html) and the [PECL memcached](#installing-pecl-memcached) extension.
 2. Take your Drupal site offline
 3. Download and install the Drupal's ["Memcache API and Integration"](https://www.drupal.org/node/108642/release) module
 4. To make Memcached the default cache class, edit your site's `settings.php` file, as shown in the example below
 5. Take your Drupal site back online

Your `settings.php` should resemble the following snippet:
	
	$conf['cache_backends'][] = 'sites/all/modules/memcache/memcache.inc';
	// The 'cache_form' bin must be assigned no non-volatile storage.
	$conf['cache_class_cache_form'] = 'DrupalDatabaseCache';
	$conf['cache_default_class'] = 'MemCacheDrupal';

	$conf['memcache_key_prefix'] = 'default';
	$conf['memcache_servers'] = array( 'hostname:port' => 'default' ); 
	$conf['memcache_sasl_auth'] = array(	'user' => 'username', 'password' => 'password' );
		
 - In line 6, the value should be your bucket's endpoint
 - In line 7, the first value should be your bucket's username
 - In line 7, the second value should be your bucket's password
	
## Using Multiple Memcached Buckets ##
Although a single Memcached bucket can be used for all of your site's needs, it is possible to use multiple buckets via the bins mechanism. The following snippet demonstrates the use of two Memcached buckets - the first for all data and the second for users:

	$conf['memcache_servers'] = array(
		'hostname1:port1' => 'default',
		'hostname2:port2' => 'users'
	);
	$conf['memcache_bins'] = array(
		'cache' => 'default',
		'users' => 'users'
	);

# Using Memcached with Joomla #
To use Memcached as a cache for your Joomla site follow these steps:

 1. Make sure that your Joomla server is installed with the [libmemcached](http://libmemcached.org/libMemcached.html) and the [PECL memcached](#installing-pecl-memcached) extension.
 2. Log into Joomla's administration console and navigate to the **Site -> Global Configuration -> System** tab
 3. Under **Cache Settings**, set the following:
	 4. *Cache:* On-Progressive
	 5. *Handler:* Memcache
	 6. *Server:* the hostname of your bucket
	 7. *Server port:* the port number of your bucket
 8. Click **Save**
 
# Using Memcached with Magento #
Magento can use Memcached both as a cache and as a session storage. We recommend you use a different Memcached bucket for each purpose.

To use Memcached as a cache and as a session storage for your Magento site, follow these steps:

1. Make sure that your Magento server is installed with [libmemcached](http://libmemcached.org/libMemcached.html) and the [PECL memcached](#installing-pecl-memcached) extension.
2. Log into Magento's administration console and set default value of the *Cookie Lifetime* in the **Magento Session Cookie Management** to *31536000* in order to comply with following Memcached  protocol specification about [expiration times](https://github.com/memcached/memcached/blob/master/doc/protocol.txt#L79):

	> Some commands involve a client sending some kind of expiration time
	> (relative to an item or to an operation requested by the client) to
	> the server. In all such cases, the actual value sent may either be
	> Unix time (number of seconds since January 1, 1970, as a 32-bit
	> value), or a number of seconds starting from current time. In the
	> latter case, this number of seconds may not exceed 60*60*24*30 (number
	> of seconds in 30 days); if the number sent by a client is larger than
	> that, the server will consider it to be real Unix time value rather
	> than an offset from current time.

3. Edit your site's XML file, typically located at `app/etc/local.xml`, and add the following to the `<global>` block:
		
		<session_cache_limiter></session _cache_limiter>
		<session_save>
			<![CDATA[memcache]]>
		</session_save>
		<session_save_path>
			<![CDATA[tcp://hostname1:port1?persistent=1&weight=1&timeout=10&retry_interval=10]]>
		</session_save_path>
		<cache>
			<backend>
				memcached
			</backend>
			<slow_backend>
				database
			</slow_backend>
			<slow_backend_store_data>
				0
			</slow_backend_store_data>
			<memcached>
				<servers>
					<server>
						<host>
							<![CDATA[hostname2]]>
						</host>
						<port>
							<![CDATA[port2]]>
						</port>
						<persistent>
							<![CDATA[1]]>
						</persistent>
						<weight>
							<![CDATA[1]]>
						</weight>
						<timeout>
							<![CDATA[10]]>
						</timeout>
						<retry_interval>
							<![CDATA[10]]>
						</retry_interval>
						<status>
							<![CDATA[1]]>
						</status>
					</server>
				</servers>
				<compression>
					<![CDATA[0]]>
				</compression>
				<cache_dir>
					<![CDATA[]]>
				</cache_dir>
				<hashed_directory_level>
					<![CDATA[]]>
				</hashed_directory_level>
				<hashed_directory_umask>
					<![CDATA[]]>
				</hashed_directory_umask>
				<file_name_prefix>
					<![CDATA[]]>
				</file_name_prefix>
			</memcached>
		</cache>
To modify these settings so that your Memcached buckets are used:
 - Replace `hostname1` with the hostname of your sessions bucket
 - Replace `port1` with the port of your sessions bucket
 - Replace `hostname2` with the hostname of your cache bucket
 - Replace `port2` with the port of your cache bucket
