# Use at your own risk!
**The script hasn't been battle tested yet and can contain bugs!**
### Usage
- Clone the repository
```bash
git clone https://github.com/Astatoru/remnackup.git
```
- Make the script executable
```bash
chmod +x /path/to/remnackup
```
- Add your current user to the `docker` group so you can run Docker without root (optional)
```bash
sudo gpasswd -a username docker
```
- Create symlink to the script inside `/usr/local/bin` directory so you can use `remnackup` without specifying full path to it (optional)
```bash
sudo ln -s /full/path/to/remnackup /usr/local/bin/remnackup
```
### Options

| Option             | Description                                                                           | Default Value           |
| ------------------ | ------------------------------------------------------------------------------------- | ----------------------- |
| `--help`           | Show help message                                                                     | -                       |
| `--root`           | Allow running the script as root or sudo (not recommended)                            | -                       |
| `--all`            | An alternative for using the --backupdb, --backupconfig, and --github options simultaneously | -                       |
| `--backupdb`       | Backup Remnawave's database                                                           | -                       |
| `--dbuser`         | Database user (check yours in `.env`)                                                 | `postgres`              |
| `--dumppath`       | Path where to dump database                                                           | Current directory       |
| `--dumpname`       | Database dump file's name                                                             | `database-$DATE.dump`   |
| `--dumparchpath`   | Path where to create .tar.gz archive with database dump                               | Current directory       |
| `--dumparchname`   | Name of the archive with database dump                                                | `database-$DATE.tar.gz` |
| `--container`      | Name of the docker container responsible for Remnawave's database                     | `remnawave-db`          |
| `--backupconfig`   | Backup Remnawave's config                                                             | -                       |
| `--configpath`     | Path to config                                                                        | `/opt/remnawave`        |
| `--configarchpath` | Path where to create .tar.gz archive with config                                      | Current directory       |
| `--configarchname` | Name of the archive with config backup                                                | `config-$DATE.tar.gz`   |
| `--github`         | Upload archive(s) with backup to Github repository                                    | -                       |
| `--repository`     | Path to your repository                                                               | -                       |
### Examples
- Backup Remnawave's database to the current directory
```bash
./remnackup --backupdb
```
- Backup everything and push to the remote repository (with a custom path to config)
```bash
./remnackup --all --configpath="/path/to/config" --repository="/path/to/repository"
```
- Cronjob that backs up everything everyday at 00:00
```bash
0 0 * * * /path/to/remnackup --all --repository="/path/to/repository"
```
### Usage with Github
If you plan on backing up your archives to GitHub, you should do some preparation
- Create your own repository
- If you want, you can login into your Github account on your VPS and entierly skip fine-grained token setup, but with token it's way more secure
- Create a fine-grained token:
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
sudo apt install gh
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
- Configure your `~/.gitconfig` (most likely not needed)
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
### How to restore the database
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
- Restore the database
    - Replace `--user "postgres"` and `--dbname "postgres"` according to your `.env` file (`postgres` values are default)
```bash
docker exec remnawave-db psql --user "postgres" --dbname "postgres" < /path/to/database-$DATE.dump
```
