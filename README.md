# SXTCLI_Wrapper_Functions
This is a collection of shell wrapper functions for the Space and Time CLI.  It uses environment variables, defaults, and native OS capabilities to dramatically simplify CLI commands.  

&nbsp;
# Requirements
This is a wrapper around the existing SXTCLI, thus assumes you have the SXTCLI installed and working, version v0.0.5 or higher.  

For more information on installing the CLI, see the official CLI documentation:
https://docs.spaceandtime.io/docs/sxtcli

&nbsp;
# Install
This is a simple shell script that is sourced by your terminal's default startup script, so there are only two files needed: 

1) The script file itself (by operating system).  For MacOS, the file is:  `.zshsxt`
2) The .env file to store keys and other secrets (universal).  The file here is: `.env_sample`

You can clone the repo, or just download the two files.   From there, the instructions depend your Operating System.

> Regardless of the OS, **ALWAYS BACK UP ANY CONFIGURATION FILES BEFORE CHANGING!**



## MacOS
Open a new TERMINAL window (aka zsh), navigate to the folder containing the two files above.  Note, both files begin with a period, which makes them 'hidden' by the OS. To confirm you're in the correct folder, type in the terminal `ls -a` and you should see your two files. Once you've confirmed your in the correct folder, enter the commands below.

1) Create a dated backup of config file (in backup folder), then print to terminal when done to verify:
    ```shell
    mkdir ~/.zshrc_bkup
    cp ~/.zshrc ~/.zshrc_bkup/.zshrc_$(date +"%Y%m%d%H%M%S")
    ls -al ~/.zshrc_bkup/
    ```

2) Copy over new functions to a new `.sxtshell` hidden folder, then print to terminal when done to verify:
    ```shell
    mkdir ~/.sxtshell
    cp .env_sample ~/.sxtshell/.env
    cp .zshsxt ~/.sxtshell/.zshsxt
    ls -al ~/.sxtshell
    ```

3) Add a line to your config file (`.zshrc`) to load the new functions from now on, then print to terminal when done to verify:
    ```shell
    echo "source ~/.sxtshell/.zshsxt" >> ~/.zshrc
    tail ~/.zshrc
    source ~/.zshrc
    ```
 
4) This will open the credential file, so you can add YOUR Space and Time credentials.  Save and close when done:
    ```shell
    open -a TextEdit ~/.sxtshell/.env
    ```
    
4) Test it out!
  - Test the install of these tools by entering the command:
    ```shell
    sxtversion
    ```
    Which should return something like:
    > sxtcli version: Version: 0.0.5
    > sxt_shortcuts:  Version: 2.0
    > for more information, see:
    >   https://github.com/SxT-Community/SXTCLI_Shell_Functions

    If that ran successfully, then the install was successful!

  - To do an end-to-end test (including running queries), make sure the `.env` file has your own credentials and enter:
    ```shell
    sxttest
    ```
    Note, it will fail on running queries if you've not added your credentials to the `.env` file.


## Other Operating Systems
As of right now, MacOS is the only OS on which this has been tested.  Probably it'll work on Linux, probably on Windows running WINE or WSL (Linux emulators). If/when demand is appropriate, we'll get back to supporting Windows natively. If someone is interested in a small weekend project, we'd love to accept PRs for extending to Windows!

&nbsp;
# Usage
This is a wrapper around the existing SXTCLI, so all functions from the CLI are still available.  This set of functions only simplify frequent tasks, allowing frequent users to move more quickly.   It primarily does this by presetting environment variables (envars for short, loaded from the .env file) and using those as defaults to common commands.  

## Commands: 

&nbsp;
- `sxtcli`
    - A simple alias for your SXTCLI jar file.  This is the same as the 'creating an alias' step in the official CLI documentation: https://docs.spaceandtime.io/docs/sxtcli  (i.e., installing this wrapper means not having to perform that step).

- `sxtlogin` 
    - Description: Loads the `.env` file and logs into the Space and Time network using those USER envars.  The `.env` file is reloaded every time, thus overwriting any changes made to the envars below since it's last run.  
    - Parameters: None.   
    - Required envars: `API_URL`, `USERID`, `USER_PUBLIC_KEY`, `USER_PRIVATE_KEY` (all set via `.env`)
    - Output envars: 
        - `ACCESS_TOKEN` - the actual access token (aka SXT Session Key)
        - `ACCESS_TOKEN_TIME` - the time the access token was issued (access tokens are only valid for 25mins)
    - Output to Terminal: the access token string (for easy copy)
    
&nbsp;
- `sxtlogin_as`
    - Same as `sxtlogin` except it does not reload the `.env` file.  This allows you the opportunity to set the envars manually (still no parameters accepted).

&nbsp;
- `sxtd*l`
    - Description: Runs the query provided on Space and Time network.  There are four flavors of this function, with a slightly different function name, but each with the same basic behavior, inputs, and outputs: 
        - `sxtdql` - executes SELECT statements
        - `sxtdml` - executes INSERTS, DELETES, UPDATES, etc.
        - `sxtddl` - executes CREATES, DROPS, etc.
        - `sxtsql` - executes arbitrary query, of any the above type
    - Parameters:
        - SQL: query text to execute, i.e., `SELECT * FROM Schema.TableName` or `INSERT INTO $RESOURCEID`, or $SQLSTRING
        - RESOURCEID: name of the table or view, i.e., `Schema.TableName`.  Can be omitted in `sxtsql`, although best practice is to include.
        - FORMAT: terminal return format, either TABLE (default for `sxtdql`) or JSON (default for all others)
        - These three parameters are order-dependent, but can be omitted ONLY IF an envar of the same name has been set.  i.e., assuming all the above envars are set, all below signature configurations are valid:
            - `sxtdql`  
            - `sxtdql "JSON"`  
            - `sxtdql $SQL "TABLE"`  
            - `sxtdql $SQL $RESOURCEID`  
            - `sxtdql $SQL $RESOURCEID "JSON"`
    - Required envars: `API_URL`, `ACCESS_TOKEN`, `SQL`
    - Conditionally Required envars: `BISCUIT` (if private table, or for all `sxtddl`), `RESOURCEID` (required for all except `sxtsql`)
    - Optional envars: `FORMAT` 
    - Output envars: 
        - `LAST_SQL` - the last SQL executed
    - Output to Terminal: a mult-line output, including
        - a visual horizontal break
        - the SQL being submitted (including any substitutions)
        - the query result, in the requested FORMAT (JSON or TABLE)
        - the total runtime
    
&nbsp;    
- `sxtversion` 
    - Description: Prints the version of the SXTCLI and this wrapper to the terminal.  No other inputs or outputs.

&nbsp;    
- `sxtdump` 
    - Description: Prints all envars specifically used by this wrapper to the terminal, along with values.  For sensitive keys, only the first 6 characters are displayed, and only the first 20 characters for access tokens and biscuits.  Also prints out the URL to this repo.  No other inputs or outputs. 

&nbsp;    
- `sxttest` 
    - Description: A quick-test of all the functions above. It will call (in order):
        - `sxtlogin` which internally calls `.env`, `sxtcli`, and `sxtlogin_as`
        - `sxtsql` and count all blocks created on Ethereum yesterday
        - `sxtdump` which internally calls `sxtversion`
