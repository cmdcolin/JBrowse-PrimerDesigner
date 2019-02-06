# JBrowse-PrimerDesigner

## About

PrimerDesigner plugin for JBrowse

Written by Nathan Liles (nml5566@gmail.com)

Updated by Colin Diesh (colin.diesh@gmail.com) to work with newer versions of JBrowse in Jan 2019


PrimerDesigner displays primers for a highlighted region's DNA sequence.

This plugin is very alpha.
## Note

This includes several small changes that are important compared to the original fork, if you'd like to see the changes see https://github.com/nml5566/JBrowse-PrimerDesigner/compare/nml5566:master...cmdcolin:master?w=1

Not all of these are strictly necessary apart from changes to main.js in my code, but could help your setup in other cases


## Possible setup instructions

Step 1: `sudo apt install libbio-primerdesigner-perl`

Step 2: Download primer3 1.1.4 as noted in the README of the original JBrowse PrimerDesigner. This means downloading from sourceforge. It is probably possible to make it work with newer version but this should be ok https://sourceforge.net/projects/primer3/files/primer3/1.1.4/

Step 3: `tar xf primer3-1.1.4.tar.gz` and then go to the src dir for primer3 and run `make`. Then manually copy primer3_core to the bin dir and name it primer3 e.g. `sudo cp primer3_core /usr/bin/primer3`. **This was especially important for my fork because the bin/primer-designer.pl was changed to remove configurability of the BINPATH due to errors I was receiving**

Step 4: Possible modify your code in main.js to use a hardcoded url e.g. `var url = 'http://localhost/cgi-bin/bin/primer-designer.pl';` instead of `var url = this._makeURL('/bin/primer-designer.pl');` since I put the primer-designer in a different directory namely /usr/bin/cgi-bin which mapped to http://localhost/cgi-bin/bin/primer-designer.pl on my server

Step 5: Copy the PrimerDesigner/ directory to the JBrowse plugins directory and add it to the config, see http://jbrowse.org/docs/plugins.html#installing-plugins for example

Step 6: Enable CGI on apache, I did this by adding the following to the top of /etc/apache2/sites-enabled/000-default.conf

```
ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
<Directory "/usr/lib/cgi-bin">
AllowOverride None
Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
Order allow,deny
Allow from all
</Directory>
```

Step 7: Copy the PrimerDesigner/bin/primer-designer.pl to /usr/lib/cgi-bin/bin/primer-designer.pl, this is the cgi-bin on my apache installation

Step 8: Create a tmp dir for the primer-designer.pl script, for me this was in /usr/bin/cgi-bin/tmp and made it writable or owned by apache user. If you don't know you can just make it world r/w e.g. chmod 777 just know that that's not security recommendation :)

Step 9: I symlinked the tmp dir `ln -s /usr/lib/cgi-bin/tmp /var/www/html/tmp`

Step 10: Re-run ./setup.sh to recompile the plugin

## Notes

As you can see, this was not easy. It is fairly tedious and error prone in fact. It probably makes sense to re-do it at some point but let me know if these instructions work


## Screenshot

![virtualbox_ubuntu5_20_01_2019_21_56_59](https://user-images.githubusercontent.com/6511937/51450327-58e22f80-1cfe-11e9-872c-6134eda2c934.png)


## Tested on


    primer3 1.1.4
    Bio::PrimerDesigner installed via apt install libbio-primerdesigner-perl
    perl 5.26
    Ubuntu 18.10


## Nathan's setup instructions

My instructions above worked for me. You can also try Nathan Liles setup instructions below

Compared with Nathan's setup instructions, the main different is

- I use my /usr/lib/cgi-bin directory instead of inside the jbrowse directory
- I install Bio::PrimerDesigner from apt

### Prerequisites

Bio::PrimerDesigner is a perl module frontend for primer3.
It can be installed via CPAN:

    sudo cpan Bio::PrimerDesigner

##### primer3 (1.14 or older)
This legacy requirement is due to Bio::PrimerDesigner, which currently isn't
compatible with anything newer.

The older versions can be found
[here](http://primer3.sourceforge.net/releases.php)

*Note: if primer3 is installed anywhere other than the default $PATH, you'll
need to change the BINPATH constant in PrimerDesigner/bin/primer-designer.pl
to reflect that.


### Installation
First copy the PrimerDesigner/ folder into your JBrowse plugins/ directory
    cp -r PrimerDesigner/ /your/jbrowse/path/plugins

Next, change permissions on a couple of paths so Apache can use them:

*Note: Make sure to substitute $APACHE_USER with the appropriate Apache user for your
OS (www-data for Linux & _www for OSX)

    cd /your/jbrowse/path/plugins/PrimerDesigner

    sudo chmod 764 bin/primer-designer.pl
    sudo chown $APACHE_USER bin/primer-designer.pl

    sudo chmod 764 tmp
    sudo chown $APACHE_USER tmp

Then, add the following to Apache's httpd.conf:

    <Directory /your/jbrowse/path/plugins/PrimerDesigner/bin>
	Options +ExecCGI
	AddHandler cgi-script .pl
    </Directory>

Make sure to reboot Apache after the change

Finally, edit JBrowse's jbrowse_conf.json and add PrimerDesigner to the plugin
list:
    plugins: [ 'PrimerDesigner' ],

If everything has gone well, you should see a PrimerDesigner pulldown option
in the menu
