# Saving PHP sessions in Postgres Database.

> *Disclaimer: Storing sessions in PHP via files(default) and other caching systems like Memcache / memcached / Redis is definitely faster but there was this use case where I had to use RDBMS for storing sessions and hence this. Use this only when it is a strict requirement to save sessions in a relational database* 

## Single file (PGSessions.php)

## Requirements:

* PostgreSQL 9.5+ [Dependency on using UPSERT]
* PHP 7.0+ (Might be compatible with 5.4+ but not tested)
* PDO for database operations
* Create "sessions" database as provided below.

	    CREATE TABLE "sessions" (
	    "id" TEXT NOT NULL UNIQUE,
	    "last_updated" BIGINT NOT NULL,
	    "expiry" bigint NOT NULL,
	    "data" TEXT NOT NULL);
	    CREATE INDEX "valid_sessions" ON "sessions"("id");
	    CREATE INDEX "nonexpired_sessions" ON "sessions"("id","expiry");

Steps to use:
Define required constants with proper details.
		

            SESSION_DURATION    -> Expiry of sessions in number of seconds.
            PG_HOST             -> Postgres HOST.
            PG_PORT             -> Port on which Postgres Listen.
            PG_DB               -> Database in which "sessions" database is created.
            PG_USER             -> User with which to login to database.
            PG_PASSWD           -> Password of the database user used to authenticate.

## Demo 

    require_once 'PGSessions.php'; //Or include it via any of custom autoloader if you have one
    
    define('PG_HOST','127.0.0.1');
    define('PG_PORT',5432);
    define('PG_DB','postgres_db');
    define('PG_USER','postgres_user');
    define('PG_PASSWD','users_password');
    define('SESSION_DURATION',86400); //Ensure SESSION_DURATION is less than a months' time in seconds else, PHP takes it as timestamp of session's expiry.
    
    use \PGSessions\PGSessions;
    $sessions_handler = new PGSessions(NULL,'Session_Name');
    session_set_save_handler($sessions_handler, true);
    // Specify name of your session below (optional), if not specified earlier (while initializing PGSessions Object)
    session_name('MySessionName'); 
    //Rest of your session definitions
    session_start();
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


