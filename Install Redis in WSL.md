To install Redis in Windows Subsystem for Linux (WSL), follow these steps:

# 1. Open WSL
	. Launch your installed Linux distribution (e.g., Ubuntu).
	. Update the package manager:

sudo apt update && sudo apt upgrade -y

# 2. Install Redis using apt:

sudo apt install redis-server -y
	
# 3. Verify the installation:

redis-server --version

# 4. Start Redis

sudo redis-server /etc/redis/redis.conf

# 5. Test Redis: Open a new terminal window in WSL and run:


redis-cli

Then, type:

PING

If everything works, Redis will reply with:

PONG

# 6. Configure redis

## 1) Open the configuration file:

sudo vi /etc/redis/redis.conf

## 2) Find the bind directive and change it to:

bind 0.0.0.0

## 3) Optionally, also disable protected mode (which restricts external connections for security reasons) by setting:

protected-mode no

## 4) Restart Redis to apply the changes:

sudo systemctl restart redis-server

## 5) Verify with netstat again:

sudo netstat -tuln | grep 6379

tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN

tcp6       0      0 ::1:6379                :::*                    LISTEN

# 7. In windows，use RedisInsight to check the connection with Redis in WSL.

