# Use at your own risk!
**The script hasn't been battle tested yet!**
### Usage
- Clone the repository
```bash
git clone https://github.com/Astatoru/remnackup.git
```
- Make the script executable
```bash
chmod +x /path/to/remnackup
```
- Add your current user to `docker` group, so you can run docker without root (optional)
```bash
sudo gpasswd -a username docker
```
- Follow the instructions below
### Variables
`ON` and `OFF` values are case insensitive

| Variable       | Description                                                                             | Default Value           |
| -------------- | --------------------------------------------------------------------------------------- | ----------------------- |
| `ROOT`         | Toggle support for running the script as root or sudo (not recommended)                 | `OFF`                   |
| `BACKUPDB`     | Responsible for turning `ON` and `OFF` Remnawave's database backup                      | `OFF`                   |
| `DBUSER`       | Remnawave's database user (check yours in `.env`)                                       | `postgres`              |
| `DUMPPATH`     | Path where database dump will be created                                                | Current directory       |
| `DUMPNAME`     | Dump file name                                                                          | `database-$DATE.dump`   |
| `ARCHIVEPATH`  | Path where archive with dump file will be created                                       | Current directory       |
| `ARCHIVENAME`  | Name of the archive with dump file                                                      | `database-$DATE.tar.gz` |
| `CONTAINER`    | Name of the docker container responsible for database                                   | `remnawave-db`          |
| `BACKUPCONFIG` | Responsible for turning `ON` and `OFF` Remnawave's config backup                        | `OFF`                   |
| `CONFIGPATH`   | Path to config directory                                                                | `/opt/remnawave`        |
| `ARCHIVEPATH2` | Path where archive with config files will be created                                    | Current directory       |
| `ARCHIVENAME2` | Name of the archive with config files                                                   | `config-$DATE.tar.gz`   |
| `GITHUB`       | Responsible for turning `ON` and `OFF` ability to push your backup to Github repository | `OFF`                   |
| `REPOSITORY`   | Path to your local github repository                                                    | Empty                   |
| `ALL`          | Enable `BACKUPDB`, `BACKUPCONFIG`, `GITHUB`                                             | `OFF`                   |
### Examples
- Backup Remnawave's database to current directory
```bash
BACKUPDB=ON bash /path/to/remnackup
```
- Backup Remnawave's database and config to current directory
```bash
BACKUPDB=ON BACKUPCONFIG=ON bash /path/to/remnackup
```
- Backup everything
```bash
ALL=ON REPOSITORY=/path/to/repository bash /path/to/remnackup
```
- Cronjob that backup everything everyday at 00:00 (`crontab -e`)
```bash
0 0 * * * ALL=ON REPOSITORY=/path/to/repository bash /path/to/remnackup
```
### Usage with Github
If you plan on backing up your settings to Github, you should do some preparations
- Create your own repository
- If you want, you can login into your Github account on your VPS and entierly skip fine-grained token setup, but with token it's way more secure
- Create Fine-grained token:
	- **Account** > **Settings** > **Developer Settings**
	- **Personal access tokens** > **Fine-grained tokens** > **Generate new token**
	- Set token name and description
	- Select the repository you have just created
	- Add permissions:
		- **Contents**: Access: Read and write
	- Copy the token
- Ssh into your VPS
- Install `gh` (Package name can be different between distributions)
```bash
apt install gh
```
- Login into you github account using **fine-grained token**
```
gh auth login
```
- Select:
	- **Github.com**
	- **HTTPS**
	- **Paste an authentication token**
- Paste your token
- Configure `~/.gitconfig` (most likely not needed)
```bash
[credential "https://github.com"]
  helper = !/usr/bin/gh auth git-credential
[credential "https://gist.github.com"]
  helper = !/usr/bin/gh auth git-credential
[user]
  email = 179386674+Astatoru@users.noreply.github.com # Set yours
  name = Astatoru # Set yours
[init]
  defaultBranch = main
```
### How to restore database
- Stop Remnawave
```bash
docker compose --file /path/to/compose.yaml down
```
- Run Remnawave's database container
```bash
docker compose --file /path/to/compose.yaml up --detach remnawave-db
```
- Extract database dump from archive
```bash
tar --extract --file /path/to/database-$DATE.tar.gz
```
- Restore database
```bash
docker exec remnawave-db psql --user "postgres" --dbname "postgres" < database-$DATE.dump
```
