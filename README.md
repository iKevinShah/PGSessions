# Saving PHP sessions in Postgres Database.

> *Disclaimer: Storing sessions in PHP via files(default) and other caching systems like Memcache / memcached / Redis is definitely faster but there was this use case where I had to use RDBMS for storing sessions and hence this. Use this only when it is a strict requirement to save sessions in a relational database* 

## Single file (PGSessions.php)

## Requirements:

* PostgreSQL 9.5+ [Dependency on using UPSERT]
* PHP 7.0+ (Might be compatible with 5.4 plus but not tested)
* \PDO Connected Database Object
* Create "sessions" database as provided below.

	    CREATE TABLE "sessions" (
	    "id" TEXT NOT NULL UNIQUE,
	    "last_updated" BIGINT NOT NULL,
	    "expiry" bigint NOT NULL,
	    "data" TEXT NOT NULL);
	    CREATE INDEX "valid_sessions" ON "sessions"("id");
	    CREATE INDEX "nonexpired_sessions" ON "sessions"("id","expiry");

Steps to use:

* Pass the PDO Connected to the class constructor.
* Define SESSION_DURATION constants to set session expiry.

            SESSION_DURATION    -> Expiry of sessions in number of seconds.

## Demo 

    
    require_once 'PGSessions.php';
    
    $pdo_connection = new PDO(...);
    
    use \PGSessions\PGSessions;
    $sessions_handler = new PGSessions($pdo_connection);
    session_set_save_handler($sessions_handler, true);
    session_name('MySessionName');
    session_start();
    session_regenerate_id(true);
    
    $_SESSION['something'] = 'foo';
    $_SESSION['something_else'] = 'bar';
    echo session_id(),'<br/>',$_SESSION['something'],'<br />',$_SESSION['something_else'] ;
    /*
    Rest of your script
    .
    .
    .
    .
    .
    .
    .
    .
    .
    .
    Before ending your script
    */
    session_write_close();
    //Note that session_write_close is not REQUIRED to be called on each page since we register_shutdown_function=true (second parameter = true in  session_set_save_handler).



