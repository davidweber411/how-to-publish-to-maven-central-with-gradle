### Description

This document should guide you through the process of publishing a gradle project to the maven central repository.

Official Github
repository: <a href="https://github.com/davidweber411/how-to-publish-to-maven-central-with-gradle" target="_blank" rel="noreferrer noopener">
davidweber411/how-to-publish-to-maven-central-with-gradle</a>

### Version stack used in this guide

| Software | Version |
|----------|---------|
| Gradle   | 7.4.2   |
| GnuPG    | 2.4.0   |

### Other supported version stacks

| Key              | Value                                              |
|------------------|----------------------------------------------------|
| Date of usage    | 2025.02.02                                         |
| Gradle version   | 8.12.1                                             |
| GnuPG version    | 2.4.7                                              |
| Tested / used by | <a href="https://github.com/yingzhuo">yingzhuo</a> |

### Guide

#### Step 1: Upload your code to a VCS

Upload your code to a VCS like github.

#### Step 2: Claim your namespace in the maven central repository

Your project will have a custom package path aka group id - e. g. <code>com.yourwebsite.yourapp</code>.<br>
This must be claimed in the maven central repository.

##### 2.1: Create an account in sonatypes JIRA

Please follow this link: https://issues.sonatype.org/

During this process, a member of Sonatype will create an account on the nexus for you. The url to the nexus is
e.g. https://s01.oss.sonatype.org/.

##### 2.2: Create an issue for your project hosting

| Topic                     | Value                                                        |
|---------------------------|--------------------------------------------------------------|
| Projekt                   | Community Support - Open Source Project Repository Hosting   |
| Type                      | New Project                                                  |
| Summary                   | Global/Central open source project packages hosting          |
| Description               | There is need for a library which simplifies abc development |
| Group Id                  | The group id                                                 |
| Project URL               | e.g. https://github.com/yourname/yourapp                     |
| SCM URL                   | e.g. https://github.com/yourname/yourapp.git                 |
| Username(s)               | Usernames (your username on github)                          |
| Already synced to central | No                                                           |

##### 2.3: Wait until a human or a bot answers

Please wait ...

##### 2.4: Prove that this namespace is really yours

A bot will guide you through this process. In short:

If your namespace is a website, then you must create a DNS TXT record with your JIRA ticket id.<br>
You can lookup this process in the internet - this is not too hard.

As alternative you can set your namespace to something like this: "io.github.yourgithubusername".<br>
Then you only need to create a temporary public repository with the ticket id as name.<br>

You will be informed per email if everything is ok or if there is a problem.

#### Step 3: Create a PGP key pair

You need this for signing your code. This is mandatory because if you do not do this, no one can verify, if this code is
really your code.

##### 3.1: Install GnuPG

    https://www.gnupg.org/download/

##### 3.2: Create your public key

Go through the wizard and type in your information:

    gpg --full-generate-key

    Keytype:   RSA and RSA
    Keylength: 2048 Bit
    Validity:  Does not expire (be aware of this!)

##### 3.3: Display all your created public keys

This will display your created public key:

    gpg --list-keys

    pub   rsa2048 2023-03-27 [SC]
          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX12345678 <-(The last 8 characters are your key id)
    uid      [ ultimativ ] Firstname Lastname email@email.com
    sub   rsa2048 2023-03-27 [E]

##### 3.4: Create a private key out of your public key

In this step you will be forced to enter a password.<br>
DO NOT FORGET THIS PASSWORD. YOUR KEY WILL BE LOST FOREVER. BACKUP THIS PASSWORD.

Create your private key:

    gpg --export-secret-keys 12345678 > "C:\Users\<username>\.gnupg\secring.gpg"

Check if your private key is created:

    gpg --list-secret-keys

#### Step 4: Create a backup of your PGP key pair

Do not ignore this step.<br>
You can not recover your keys if they are messed up.<br>
Your key will live in the internet forever.<br>
If this step does not work, try as long as you need until this step works.<br>
Do not go any further without creating a backup.<br>
I recommend during both style of the backup - just for security.

##### 4.1: Hardcopy style

Create the backup like this:

    cp ~/.gnupg/pubring.gpg /path/to/backups/
    cp ~/.gnupg/secring.gpg /path/to/backups/
    cp ~/.gnupg/trustdb.gpg /path/to/backups/

Import the backup like this:

    cp /path/to/backups/*.gpg ~/.gnupg/

##### 4.2: Export/Import style

Create the backup:

    // Public keys
    gpg --export --export-options backup --output publicKeysBackup.gpg
    
    // Private keys
    gpg --export-secret-keys --export-options backup --output privateKeysBackup.gpge this:
    
    // Trust relationship database
    gpg --export-ownertrust > trustBackup.gpg

Import the backup:

    // Public keys
    gpg --import publicKeysBackup.gpg

    // Private keys
    gpg --import privateKeysBackup.gpg

    // Trust relationship database
    gpg --import-ownertrust trustBackup.gpg

    If this does not work, try this:
    gpg --edit-key email@email.com
    Enter: trust
    Enter: 5
    Enter: j/y

Check for correct import:

    gpg --list-secret-keys --keyid-format LONG

#### Step 5: Publish your public key on a public key server

Publish your key:

    gpg --keyserver keyserver.ubuntu.com --send-keys XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX12345678

Check if your key is published:

    gpg --keyserver hkp://keyserver.ubuntu.com --search-key 'email@email.com'

Additional notes and informations: https://central.sonatype.org/publish/requirements/gpg/#distributing-your-public-key

#### Step 6: Prepare your global gradle.properties for publishing

The global gradle.properties are located in <code>userHome/.gradle/gradle.properties</code>. If this file does not
exist, then create it.

The project intern gradle wrapper will look into this file too.<br>
Here you can save your password from your key etc. This should not be published. ;)

Please look in the directory <code>exampleFiles</code>.

    Please read the comments in the example file of this repository.

#### Step 7: Prepare your build.gradle for publishing

##### 7.1: Generate a security token

You must generate a security token because the old way (using the standard username and password) does not work anymore.

1. Navigate to https://s01.oss.sonatype.org/.
2. Click on your "golden" user name and navigate to "Profile".
3. Select "User Token" in the dropdown.
4. Click on "Access User Token", get the token-username and the token-password and save them in an environment variable.
5. If your IDE was open during this process, restart it - TRUST ME, DO IT.

![img.png](img.png)

##### 7.2: Example file

Please look in the directory <code>exampleFiles</code>.

    Please read the comments in the example file of this repository.

#### Step 8: Publish to OSSRH nexus with gradle publish

Execute the gradlew task <code>publish</code> to publish to the OSSRH nexus.<br>
Now you need to wait around 5-30 minutes and after that, your uploaded library will appear in the repositories tab
at https://s01.oss.sonatype.org/.

#### Step 9: Release your library

After publishing to the nexus, your library is in the state <code>open</code>.<br>
You need to set the state to <code>close</code> with the <code>close button</code>.<br>
Press the <code>refresh button</code>, because this seems to be buggy.<br>
Now the <code>release button</code> is activated.<br>
Press the <code>release button</code> for releasing your library.<br>
Your library will by synced to the maven central repository within 30 minutes.

Please read this articles (annoying, but necessary): <br>
https://central.sonatype.org/publish/release/#locate-and-examine-your-staging-repository <br>
https://central.sonatype.org/publish/publish-guide/#releasing-to-central

#### Step 10: Profit

### Error handling

#### General todo

If any error occurs, than restart at first your IDE. After that, rerun the gradle task <code>gradle publish</code>
with <code>--stacktrace</code>. It will show you hints why.

#### Caused by gpg2.exe is not found

Check your gpg version with <code>gpg --version</code>. If your version is <code>>2</code> and your environment
variable <code>PATH</code> contains the path to GnuPG (e.g. <code>C:\Program Files (x86)\gnupg\bin</code>), then check
if there is a file called <code>gpg2.exe</code>. If not, check if there is a file called <code>gpg.exe</code> and rename
it to <code>gpg2.exe</code>.

IMPORTANT: From now on, you need to use <code>gpg2</code> with parameter like <code>--version</code> in your
terminal/command prompt because there is no longer a <code>gpg.exe</code>!!!

#### 401 Content access is protected by token

This error message occurs if you try to publish to OSSRH via your username and password. This does not work anymore.
You must create and use a user token.

1. Navigate to https://s01.oss.sonatype.org/.
2. Click on your "golden" user name and navigate to "Profile".
3. Select "User Token" in the dropdown.
4. Click on "Access User Token", get the token-username and the token-password and save them in an environment variable.
5. Restart your IDE after doing that - DON'T ASK, DO IT.

Original documentation: https://central.sonatype.org/publish/generate-token/

#### The dependency is not available on maven central repository

Don't get fooled by the indexed search on https://mvnrepository.com/. This process can take around 2 days (personal
experience). If you want to test it, create a separate project, include the dependency in it, and try to load it. Or
wait until you get grey hair.

### What to do if my key was hacked

You need to revoke the key from the keyserver by publishing a revocation certificate to it. This informs others that the
key shouldn't be trusted anymore.

#### Step 1: Get the ID of your key first:

For a public key use <code>gpg --list-keys</code>, for a private key use <code>gpg --list-secret-keys</code>.
In both ways the ID of the keys are the last 8 characters.

    gpg --list-keys

    pub   rsa2048 2023-03-27 [SC]
          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX12345678 <-(The last 8 characters are your key id)
    uid      [ ultimativ ] Firstname Lastname email@email.com
    sub   rsa2048 2023-03-27 [E]

#### Step 2: Mark the key as unsecure:

1. Generate a revocation certificate:<br><code>gpg --output revoke.asc --gen-revoke 12345678</code><br>You must enter
   the password of your key during this process.
2. Revoke the key:<br><code>gpg --import revoke.asc</code><br><code>gpg --send-keys 12345678</code>
3. Export the public key:<br><code>gpg --armor --export 12345678 > public-key.asc</code>
4. Send the key to the keyserver:<br><code>gpg --send-keys 12345678</code>
5. Done!
