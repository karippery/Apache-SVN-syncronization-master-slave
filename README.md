# Apache-SVN-syncronization-master-slave
SVN REPOSITORY SERVER  AND SYNCHRONIZATION
## Install SVN on all your Debian 9 server

<pre><code class="text">
apt-get install apache2 subversion libapache2-mod-svn libsvn-dev
</code></pre>

### Create First SVN Repository

<pre><code class="text">
mkdir -p /var/lib/svn/
svnadmin create /var/lib/svn/myrepo
chown -R www-data:www-data /var/lib/svn
chmod -R 775 /var/lib/svn

</code></pre>

### Create Users for Subversion

<pre><code class="text">
touch /etc/apache2/dav_svn.passwd
htpasswd -m /etc/apache2/dav_svn.passwd admin
New password:
Re-type new password:

htpasswd -m /etc/apache2/dav_svn.passwd user1
New password:
Re-type new password:
</code></pre>

### Configure Apache with Subversion
on file  /etc/apache2/mods-enabled/dav_svn.conf


<pre><code class="xml">
<Location /svn>

   DAV svn
   SVNParentPath /var/lib/svn

   AuthType Basic
   AuthName "Subversion Repository"
   AuthUserFile /etc/apache2/dav_svn.passwd
   Require valid-user
     
</Location>
</code></pre>

restart the Apache service to apply the new configuration

<pre><code class="text">
service apache2 restart

</code></pre>

### Access Repository in Browser

[[http://***.168.56.129/svn/myrepo/]]


### Import Project to the SVN Repository

Create a new svn-templates project directory.

<pre><code class="text">
mkdir -p ~/svn-templates/{trunk,branches,tags}
</code></pre>


Add templates directory to the repository using the svn command below.

<pre><code class="text">
 svn import -m 'Initial import' ~/svn-templates/ file:///var/lib/svn/myrepo/ --username admin --password Qwe12345!
</code></pre>



nano /var/lib/svn/myrepo/hooks/pre-revprop-change on master

<pre><code class="yaml">
#!/bin/sh
REVISION=${2}
# Launch (backgrounded) sync jobs for each slave server.
svnsync copy-revprops http://**.226.179.206/svn/myrepo ${REVISION} &
svnsync copy-revprops http://**.226.179.207/svn/myrepo ${REVISION} &
exit 0

</code></pre>


nano /var/lib/svn/myrepo/hooks/post-commit on master
<pre><code class="text">
#!/bin/sh
# Launch (backgrounded) sync jobs for each slave server.
svnsync sync --username admin --password Qwe12345! http://**.226.179.206/svn/myrepo &
svnsync sync --username admin --password Qwe12345! http://**.226.179.207/svn/myrepo &

</code></pre>

chmod +x /var/lib/svn/myrepo/hooks/pre-revprop-change on master

chmod +x /var/lib/svn/myrepo/hooks/post-commit on master


<pre><code class="text">
svnsync init  --username admin --password Qwe12345!   http://**.226.179.208/svn/myrepo  --allow-non-empty   file:///var/lib/svn/myrepo/ ${REVISION}
svnsync init  --username admin --password Qwe12345!   http://**.226.179.206/svn/myrepo  --allow-non-empty   file:///var/lib/svn/myrepo/ ${REVISION}
 svnsync init  --username admin --password Qwe12345!   http://**.226.179.207/svn/myrepo  --allow-non-empty   file:///var/lib/svn/myrepo/ ${REVISION}
svnsync sync --username admin --password Qwe12345! http://**.226.179.207/svn/myrepo
svnsync sync --username admin --password Qwe12345! http://**.226.179.206/svn/myrepo
</code></pre>
