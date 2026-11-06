# Setup Journal

Notes on getting the Virtual Tools game running locally, following [README.md](README.md).

## 2026-11-06 — Initial local setup (macOS, Apple Silicon)

### Starting state
- PHP: not installed.
- MySQL: only Anaconda's MySQL 5.7 binaries (`/opt/anaconda3/bin/mysql`), with no server running. **Note:** Anaconda's `mysql` comes first on `PATH`, so use the full Homebrew paths below.
- Homebrew: installed at `/opt/homebrew`.

### 1. Install PHP and MySQL
```sh
brew install php mysql
```
Installed PHP 8.5.12 and MySQL 26.10.0. The code uses `mysqli`, which works on modern PHP. The "Legacy Password Encryption" step in the README was **not** needed.

### 2. Start MySQL
```sh
brew services start mysql        # runs in the background and starts again at login
/opt/homebrew/bin/mysqladmin -u root ping   # -> "mysqld is alive"
```

### 3. Set the root password and import the database
The password matches the default in `web_content/phpcmd/server.php` (`root` / `localsql`, db `tools_mit_museum`), so that file needed no changes.
```sh
/opt/homebrew/bin/mysql -u root -e "ALTER USER 'root'@'localhost' IDENTIFIED BY 'localsql'; CREATE DATABASE IF NOT EXISTS tools_mit_museum;"
/opt/homebrew/bin/mysql -u root -plocalsql tools_mit_museum < tools_mit_museum.sql
```
Result: 4 tables (`community_trial_list`, `responses_tool_select`, `trial_list`, `trials`) and 58 rows in `trial_list`. No import errors.

Sequel Ace isn't needed because the command-line client did the import.

### 4. Start the PHP server
```sh
cd web_content
/opt/homebrew/bin/php -S 0.0.0.0:8000
```
Open http://localhost:8000.

### 5. Verification
- Tested PHP → MySQL with `mysqli` using `server.php`: connected, 58 trials.
- `/`, `/community_levels.php`, `/phpcmd/load_trial_list.php` all return HTTP 200 with no PHP errors.

### Day-to-day usage
- MySQL keeps running as a brew service. Stop it with `brew services stop mysql`.
- To play: `cd web_content && php -S 0.0.0.0:8000`, then open http://localhost:8000.
- Other devices on the same network (e.g. a museum touchscreen) can connect at `http://<this-mac's-IP>:8000`, because the server binds to `0.0.0.0`. Find the IP with `ipconfig getifaddr en0`.
- Community level submission password (from the README): `cocosci`.

### Stopping it
Two things run separately, and each one has to be stopped on its own. Stopping them deletes nothing: levels and recorded data stay in the database.

1. **The PHP web server** (port 8000)
   - If it's running in your terminal, press **Ctrl+C** there.
   - If it's running in the background, for example started by Claude:
     ```sh
     lsof -ti :8000 | xargs kill
     ```
2. **MySQL** (port 3306)
   ```sh
   brew services stop mysql    # also stops it from starting at login
   ```

To check that both are stopped (no output means nothing is running):
```sh
lsof -nP -iTCP:8000 -sTCP:LISTEN; lsof -nP -iTCP:3306 -sTCP:LISTEN
```

To start again: `brew services start mysql`, then `cd web_content && /opt/homebrew/bin/php -S 0.0.0.0:8000`.

### Open items / notes
- Stopping Claude's background task didn't fully stop the PHP server: a leftover `php` process kept listening on port 8000 and had to be killed by PID. Always run the port check above to make sure it's actually stopped.
- _(add issues and changes here)_
