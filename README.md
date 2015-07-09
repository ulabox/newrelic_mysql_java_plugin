# Ulabox Replication Status Plugin

New Relic plugin that extends form the MySQL plugin to give more accurate information about replication in your MySQL database.


## Requirements

The requirements for running this plugin are:

- A New Relic account. Sign up for a free account [here](http://newrelic.com)
- Java Runtime (JRE) environment Version 1.6 or later
- A server running MySQL Version 5.0 or greater
- Network access to New Relic (authenticated proxies are not currently supported, but see workaround below)

**Note:** The MySQL Plugin includes the [Connector/J JDBC Driver](http://dev.mysql.com/usingmysql/java/) and it does not need to be installed separately.


## Installation

Ulabox Replication Status is installed via New Relic's NPI:

	1. Download the manifest.json from this repository.
	2. Place it in the config directory from your NPI's installation folder.
	3. Run `./npi install <guid> --untrusted`
	
    
## Configuration Information

### Configuration Files

You will need to modify two configuration files in order to set this plugin up to run.  The first (`newrelic.json`) contains configurations used by all Platform plugins (e.g. license key, logging information, proxy settings) and can be shared across your plugins.  The second (`plugin.json`) contains data specific to each plugin such as a list of hosts and port combination for what you are monitoring.  Templates for both of these files should be located in the '`config`' directory in your extracted plugin folder. 

#### Configuring the `plugin.json` file: 

The `plugin.json` file has a provided template in the `config` directory named `plugin.template.json`.  If you are installing manually, make a copy of this template file and rename it to `plugin.json` (the New Relic Platform Installer will automatically handle creation of configuration files for you).  

Below is an example of the `plugin.json` file's contents, you can add multiple objects to the "agents" array to monitor different instances:

```
    {
      "agents": [
        {
          "name" : "Production Master",
          "host" : "localhost",
          "metrics" : "status,newrelic",
          "user" : "USER_NAME_HERE",
          "passwd" : "USER_CLEAR_TEXT_PASSWORD_HERE"
        }
      ]
    }
```

**note** - Set the "name" attribute to match your MySQL databases purpose, e.g. "Production Master" as this will be used to identify that instance in the New Relic UI. 

**note** - If you used the provided [/scripts/mysql_user.sql](https://github.com/newrelic-platform/newrelic_mysql_java_plugin/blob/master/scripts/mysql_user.sql) to generate a default user and password, then you do not need to set the "user" and "passwd" attributes.

**note** - If using an externally visible IP address, the username and password fields are no longer optional. See the 'Create MySQL user (optional)' section below.

#### Configuring the `newrelic.json` file: 

The `newrelic.json` file also has a provided template in the `config` directory named `newrelic.template.json`.  If you are installing manually, make a copy of this template file and rename it to `newrelic.json` (again, the New Relic Platform Installer will automatically handle this for you).  

The `newrelic.json` is a standardized file containing configuration information that applies to any plugin (e.g. license key, logging, proxy settings), so going forward you will be able to copy a single `newrelic.json` file from one plugin to another.  Below is a list of the configuration fields that can be managed through this file:

##### Configuring your New Relic License Key

Your New Relic license key is the only required field in the `newrelic.json` file as it is used to determine what account you are reporting to.  If you do not know what your license key is, you can learn about it [here](https://newrelic.com/docs/subscriptions/license-key).

Example: 

```
{
  "license_key": "YOUR_LICENSE_KEY_HERE"
}
```

##### Logging configuration

By default Platform plugins will have their logging turned on; however, you can manage these settings with the following configurations:

`log_level` - The log level. Valid values: [`debug`, `info`, `warn`, `error`, `fatal`]. Defaults to `info`.

`log_file_name` - The log file name. Defaults to `newrelic_plugin.log`.

`log_file_path` - The log file path. Defaults to `logs`.

`log_limit_in_kbytes` - The log file limit in kilobytes. Defaults to `25600` (25 MB). If limit is set to `0`, the log file size would not be limited.

Example:

```
{
  "license_key": "YOUR_LICENSE_KEY_HERE"
  "log_level": "debug",
  "log_file_path": "/var/logs/newrelic"
}
```

##### Proxy configuration

If you are running your plugin from a machine that runs outbound traffic through a proxy, you can use the following optional configurations in your `newrelic.json` file:

`proxy_host` - The proxy host (e.g. `webcache.example.com`)

`proxy_port` - The proxy port (e.g. `8080`).  Defaults to `80` if a `proxy_host` is set

`proxy_username` - The proxy username

`proxy_password` - The proxy password

Example:

```
{
  "license_key": "YOUR_LICENSE_KEY_HERE",
  "proxy_host": "proxy.mycompany.com",
  "proxy_port": 9000
}
```


### Additional Configuration

#### Create a MySQL user (optional)

The MySQL plugin requires a MySQL user with limited privileges. To use the New Relic default, run the following SQL script located at [/scripts/mysql_user.sql](https://github.com/newrelic-platform/newrelic_mysql_java_plugin/blob/master/scripts/mysql_user.sql).

`$ mysql -uroot -p < mysql_user.sql`

This script will create the following user:

    username: newrelic
    host: localhost or 127.0.0.1
    password: *B8B274C6AF8165B631B4B517BD0ED2694909F464 (hashed value)

*You can choose to use a different MySQL user name and password. See [MYSQL.TXT](https://github.com/newrelic-platform/newrelic_mysql_java_plugin/blob/master/MYSQL.TXT) for more info.*

If your MySQL Server is bound to an externally visible IP address, both localhost and 127.0.0.1 will not be accessible via TCP as the host for the MySQL Plugin. In order for the plugin to connect, you will need to create a user for your IP address. Due to security concerns in this case, we strongly recommend **not** using the default password and instead setting it to some other value.

    CREATE USER newrelic@<INSERT_IP_ADDRESS_HERE> IDENTIFIED BY '<INSERT_HASHED_PASSWORD_HERE>';
    GRANT PROCESS, REPLICATION CLIENT ON *.* TO newrelic@<INSERT_IP_ADDRESS_HERE>;


#### Selecting metrics

The MySQL Plugin is capable of reporting different sets of metrics by configuring the 'metrics' attribute. E.g., add the 'slave' category to report replication metrics. 
*See [CATEGORIES.TXT](https://github.com/newrelic-platform/newrelic_mysql_java_plugin/blob/master/CATEGORIES.TXT) for more info.*

**Note:** The `innodb_mutex` metric category can lead to increased memory usage for the plugin if the monitored database is under a high level of contention (i.e. large numbers of active mutexes).


## Credits

The MySQL plugin was originally authored by [Ronald Bradford](http://ronaldbradford.com/) of [EffectiveMySQL](http://effectivemysql.com/). Subsequent updates and support are provided by [New Relic](http://newrelic.com/platform).
