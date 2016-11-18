# Rundeck CLI Tool

[![Build Status](https://travis-ci.org/rundeck/rundeck-cli.svg?branch=master)](https://travis-ci.org/rundeck/rundeck-cli)

This is a new CLI tool for [Rundeck](https://github.com/rundeck/rundeck).

Its goal is to replace the old CLI Tools currently included with Rundeck with a modernized,
extensible, and nicer implementation.

This project is now at 1.0.x status.

See [todos.md](https://github.com/rundeck/rundeck-cli/blob/master/todo.md)
## Requirements

Java 8

##Download

[github releases](https://github.com/rundeck/rundeck-cli/releases)

* `rd-x.y.zip`/`rd-x.y.tar` zip distribution
* `rundeck-cli-x.y-all.jar` standalone executable jar
* `rundeck-cli-x.y.noarch.rpm` rpm
* `rundeck-cli-x.y_all.deb` debian


### Yum usage

Also, in bintray: [bintray/rundeck/rundeck-rpm](https://bintray.com/rundeck/rundeck-rpm).

~~~{.sh}
$ wget https://bintray.com/rundeck/rundeck-rpm/rpm -O bintray.repo
$ sudo mv bintray.repo /etc/yum.repos.d/
$ yum install rundeck-cli
~~~

optional: enable all gpg checks:

~~~{.sh}
$ sed -i.bak s/gpgcheck=0/gpgcheck=1/ /etc/yum.repos.d/bintray.repo
$ echo "gpgkey=https://bintray.com/user/downloadSubjectPublicKey?username=bintray" >> /etc/yum.repos.d/bintray.repo
$ rpm --import http://rundeck.org/keys/BUILD-GPG-KEY-Rundeck.org.key 
~~~

optional: enable only rpm gpg checks:

~~~{.sh}
$ sed -i.bak s/^gpgcheck=0/gpgcheck=1/ /etc/yum.repos.d/bintray.repo
$ echo "gpgkey=http://rundeck.org/keys/BUILD-GPG-KEY-Rundeck.org.key" >> /etc/yum.repos.d/bintray.repo 
~~~

### Debian usage

(todo...)

On bintray: [bintray/rundeck/rundeck-deb](https://bintray.com/rundeck/rundeck-deb).

## Usage


### Configuration

Define access credentials as user/password or Token value:

	export RUNDECK_URL=http://rundeck:4440

	export RUNDECK_TOKEN=....

	# or

	export RUNDECK_USER=username
	export RUNDECK_PASSWORD=password

Define a specific API version to use, by using the complete API base:

	export RUNDECK_URL=http://rundeck:4440/api/12

All requests will be made using that API version.

**Bypass an external URL**:

If your Rundeck server has a different *external URL* than the one you are accessing,
you can tell the `rd` tool to treat redirects to that external URL as
if they were to the original URL you specified.

	export RUNDECK_URL=http://internal-rundeck:4440/rundeck
	export RUNDECK_BYPASS_URL=https://rundeck.mycompany.com

This will rewrite any redirect to `https://rundeck.mycompany.com/blah`
as `http://internal-rundeck:4440/rundeck/blah`.

Note: if you include the API version in your `RUNDECK_URL`, e.g. `http://internal-rundeck:4440/rundeck/api/12` then
the `RUNDECK_BYPASS_URL` will be replaced by `http://internal-rundeck:4440/rundeck`.

**HTTP/connect timeout**

Use `RD_HTTP_TIMEOUT` env var:

	# 30 second timeout
	export RD_HTTP_TIMEOUT=30

Note: if the timeout seems longer than you specify, it is because the "connection retry" is set to true
by default.

**Connection Retry**

Retry in case of recoverable connection issue (e.g. failure to connect):

Use `RD_CONNECT_RETRY` (default `true`):

	# don't retry
	export RD_CONNECT_RETRY=false

**Debug HTTP**

Use the `DEBUG` env var to turn on HTTP debugging:

	export DEBUG=1 # basic http request debug
	export DEBUG=2 # http headers
	export DEBUG=3 # http body

### Scripting

Specifying formatted output for Job and Execution lists:

	# output only id and href
	rd jobs list -p myproject -% "%id %href"

	# output id and execution status
	rd executions query -p myproject -O 3d -% "%id %status"

### Json and Yaml support

(currently work in progress)

JSON and YAML output:

	RD_FORMAT=json rd jobs list
	RD_FORMAT=yaml rd jobs list


### Running

Install `rd-0.x.y.zip`

	rd
	├── bin
	│   ├── rd
	│   └── rd.bat
	└── lib
	    ├── ....jar

The `rd` binary provides top level commands:

	$ rd help
	Available commands: [projects, executions, jobs, run, adhoc]
	...

The functionality of previous "rd-*" tools (from Rundeck) is no split into subcommands.

## Commands

Available commands:

	   adhoc      - Dispatch adhoc COMMAND to matching nodes
	   executions - List running executions, attach and follow their output, or kill them
	   jobs       - List and manage Jobs
	   keys       - Manage Keys via the Key Storage Facility.
	   projects   - List and manage projects
	   run        - Run a Job
	   scheduler  - View scheduler information
	   system     - View system information
	   tokens     - Create, and manage tokens

	Use "rd [command] help" to get help on any command.

### adhoc

Dispatch adhoc COMMAND to matching nodes.

	Usage: adhoc options -- COMMAND...
		[--filter -F value] : A node filter string
		[--follow -f] : Follow execution output as it runs
		[--keepgoing -K] : Keep going when an error occurs on multiple dispatch
		[--progress -r] : Do not echo log text, just an indicator that output is being received.
		--project -p value : Project name
		[--quiet -q] : Echo no output, just wait until the execution completes.
		[--restart -t] : Restart from the beginning
		[--script -s value] : Dispatch specified script file
		[--stdin -S] : Execute input read from STDIN
		[--tail -T value] : Number of lines to tail from the end, default: 1
		[--threadcount -C value] : Dispatch execution to Nodes using COUNT threads
		[--url -u value] : Download a URL and dispatch it as a script

### executions

List running executions, attach and follow their output, or kill them.


	Available commands:

	   delete     - Delete an execution by ID
	   deletebulk - Find and delete executions in a project
	   follow     - Follow the output of an execution
	   info       - List all running executions for a project
	   kill       - Attempt to kill an execution by ID
	   list       - List all running executions for a project
	   query      - Query previous executions for a project

#### execution query

Query previous executions for a project.

	Usage: query options
		[--adhoconly -A] : Adhoc executions only
		[--xgroup value] : Group or partial group path to exclude, "-" means top-level jobs only
		[--xgroupexact value] : Exact group path to exclude, "-" means top-level jobs only
		[--xnameexact value] : Exclude Exact Job Name Filter, exclude any name that is equal to this value
		[--xname value] : Exclude Job Name Filter, exclude any name that matches this value
		[--xjobids -x value...] : Job ID list to exclude
		[--xjobs -X value...] : List of Full job group and name to exclude.
		[--group -g value] : Group or partial group path to include, "-" means top-level jobs only
		[--groupexact -G value] : Exact group path to include, "-" means top-level jobs only
		[--jobonly -J] : Job executions only
		[--nameexact -N value] : Exact Job Name Filter, include any name that is equal to this value
		[--name -n value] : Job Name Filter, include any name that matches this value
		[--jobids -i value...] : Job ID list to include
		[--jobs -j value...] : List of Full job group and name to include.
		[--max -m value] : Maximum number of results to retrieve at once.
		[--offset -o value] : First result offset to receive.
		[--older -O value] : Get executions older than specified time. e.g. "3m" (3 months).
	Use: h,n,s,d,w,m,y (hour,minute,second,day,week,month,year)
		[--outformat -% value] : Output format specifier for execution data. You can use "%key" where key is one of:id, project, description, argstring, permalink, href, status, job, user, serverUUID, dateStarted, dateEnded, successfulNodes, failedNodes. E.g. "%id %href"
		--project -p value : Project name
		[--recent -d value] : Get executions newer than specified time. e.g. "3m" (3 months).
	Use: h,n,s,d,w,m,y (hour,minute,second,day,week,month,year)
		[--status -s value] : Status filter, one of: running,succeeded,failed,aborted
		[--user -u value] : User filter
		[--verbose -v] : Extended verbose output

### jobs

List and manage Jobs.


	Available commands:

	   info  - Get info about a Job by ID (API v18)
	   list  - List jobs found in a project, or download Job definitions (-f)
	   load  - Load Job definitions from a file in XML or YAML format
	   purge - Delete jobs matching the query parameters

### keys

Manage Keys via the Key Storage Facility.
Specify the path using -p/--path, or as the last argument to the command.


	Available commands:

	   create - Create a new key entry
	   delete - Delete the key at the given path
	   get    - Get the contents of a public key
	   info   - Get metadata about the given path
	   list   - List the keys and directories at a given path, or at the root by default
	   update - Update an existing key entry

### projects

List and manage projects.

	Available commands:

	   acls   - Manage Project ACLs
	   create - Create a project
	   delete - Delete a project
	   list   - List all projects
	   readme - Manage Project readme
	   scm    - Manage Project SCM

### run

Run a Job.

	Usage: run [options] -- -ARG VAL -ARG2 VAL...
		[--filter -F value] : A node filter string
		[--follow -f] : Follow execution output as it runs
		[--id -i value] : Run the Job with this IDENTIFIER
		[--job -j value] : Job job (group and name). Run a Job specified by Job name and group. eg: 'group/name'.
		[--logevel -l /(verbose|info|warning|error)/] : Run the command using the specified LEVEL. LEVEL can be verbose, info, warning, error.
		[--progress -r] : Do not echo log text, just an indicator that output is being received.
		[--project -p value] : Project name
		[--quiet -q] : Echo no output, just wait until the execution completes.
		[--restart -t] : Restart from the beginning
		[--tail -T value] : Number of lines to tail from the end, default: 1
		[--user -u value] : A username to run the job as, (runAs access required).

### scheduler

View scheduler information

List jobs for the current target server, or a specified server.

	Usage: jobs [options]
		[--uuid -u value] : Server UUID to query, or blank to select the target server

### system

View system information

Print system information and stats.