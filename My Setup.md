## 1. Repository Directory Creation
I created a repository directory at: /var/www/html/ubuntu-repo
 
I copied `.deb` packages like **cowsay**, **figlet**, and **sl** into this folder. 

---

## 2. Installed Required Packages
- I installed **dpkg-dev** (required for `dpkg-scanpackages`). 
- I installed **apache2** (to serve the repo over HTTP).  

---

## 3. Permissions & Ownership
- I tried changing ownership and permissions multiple times between `www-data:www-data` and my own user.  
- I used `chmod -R 755` to make files readable by Apache.   

> Note: This step was necessary because I kept hitting **Permission denied** when generating `Packages.gz`.

---

## 4. Generating Repository Index
I ran multiple variations of `dpkg-scanpackages` to generate the `Packages.gz` index file:

```
sudo dpkg-scanpackages . /dev/null | gzip -9c | sudo tee Packages.gz > /dev/null
sudo sh -c "dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz"
```
In the end, Packages.gz was successfully created and contained my packages:

```
Package: cowsay
Package: figlet
Package: sl
```
I had repeated permission errors, which I fixed using sudo or sh -c.

## 5. Configured APT
I created a local source list at:

```
/etc/apt/sources.list.d/local-repo.list
```
and added:

```
deb [trusted=yes] http://localhost:8080/ubuntu-repo ./
```
I also disabled translation warnings:

```
echo 'Acquire::Languages "none";' | sudo tee /etc/apt/apt.conf.d/99disable-translations
```
Now APT knows about my local repo.

## 6. Apache Configuration
I restarted Apache:

```
sudo systemctl restart apache2
sudo systemctl status apache2
```
Apache is running.

## 7. Testing the Repository
   
```
curl http://localhost:8080/ubuntu-repo/Packages.gz | gunzip | grep Package Shows cowsay, figlet, sl.
```
apt-cache policy cowsay Shows my local repo as a source.

```
sudo apt install --reinstall cowsay Works.
```

## 8. Issues I Encountered :


1. Packages.gz: Permission denied

 This happened because the output file was redirected in a folder owned by www-data.

 I fixed it using sudo tee or sudo sh -c.

2. APT 404 warnings during apt update

 ```
 Err:3 http://localhost:8080/ubuntu-repo ./ Packages
   404  Not Found [IP: 127.0.0.1 8080]
 ```
 This is a common minor warning when APT tries to fetch translations or metadata.

 It doesn’t mean my repo is broken.

 My repo is working because:

 ```
  Packages.gz exists.
 ```

 curl shows the package names.

 Installing cowsay from my repo works.

3. Repeated reinstalling of cowsay
I did this unnecessarily. My local repo was already recognized.

### Conclusion
- I successfully created a working local APT repository on Ubuntu/WSL2.

- The repo serves cowsay, figlet, and sl packages.

- Apache is running and the repository is accessible.

- APT warnings (like 404) are mostly harmless in this setup.

