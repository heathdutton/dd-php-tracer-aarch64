# dd-php-tracer-aarch64
# https://stathers.net/2020/03/17/datadog-php-tracer-aarch64.html

```
mkdir -p /tmp/dd-php-tracer-aarch64
cd /tmp/dd-php-tracer-aarch64
# download latest release tarball
curl -s https://api.github.com/repos/DataDog/dd-trace-php/releases/latest | grep "tarball_url" | cut -d : -f 2,3 | tr -d \", | wget -qi -
# untar it
tar -xvf *
# Jump in and compile
cd DataDog*
make
# Should now be here:
ls ./tmp/build_extension/modules/ddtrace.so
mkdir -p /tmp/dd-php-tracer-aarch64-rpm
cd /tmp/dd-php-tracer-aarch64-rpm
curl -s https://api.github.com/repos/DataDog/dd-trace-php/releases/latest | grep "x86_64.rpm" | cut -d : -f 2,3 | tr -d \", | wget -qi -
# Rename the rpm
rename x86_64 aarch64 *.rpm
# Get rpmrebuild
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum install -y rpmrebuild
# Force it to aarch64
sudo sed -i 's/pac_arch=$( spec_query qf_spec_arch )/pac_arch="aarch64"/' /usr/lib/rpmrebuild/rpmrebuild.sh

### MANUAL ###

# Edit the RPM 
# replace any instance of x86_64 with aarch64. This vim command will do it for you: :%s/x86.64/aarch64/
# Delete lines with: (use Esc, dd)
# %attr(0755, root, root) "/opt/datadog-php/extensions/ddtrace-
# Leave in one line sating:
# %attr(0755, root, root) "/opt/datadog-php/extensions/ddtrace.so"
sudo rpmrebuild -enp --change-files "/bin/bash" *.rpm
# Quit with :wq, and say "Yes" to continue

### AUTO ###

# Delete pre-existing .so
rm -f /root/.tmp/rpmrebuild.*/work/root/opt/datadog-php/extensions/ddtrace*
# copy in our new .so
cp /tmp/dd-php-tracer-aarch64/DataDog*/tmp/build_extension/modules/ddtrace.so /root/.tmp/rpmrebuild.*/work/root/opt/datadog-php/extensions/
# Exit to finish the RPM edit
exit 0

### MANUAL ###
sudo su
cd /root/rpmbuild/RPMS/aarch64
git clone https://github.com/heathdutton/dd-php-tracer-aarch64
cp *.rpm ./dd-php-tracer-aarch64
cd dd-php-tracer-aarch64/
git add . 
git commit -m "new version"
git push
```
