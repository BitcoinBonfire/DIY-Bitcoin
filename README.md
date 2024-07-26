# DIY Financial Self Sovereignty

In my opinion, this is the current best practices for a do-it-yourself trusted Bitcoin operation to support and transact that focuses on security/anonymity and minimal third-party involvement/dependencies.

My first journey into Linux was setting this up. Took a bit of research and had fun doing it. I'm still a total noob but that's why I think this procedure is great... It's written by a noob for a noob wanting to reach the endpoint of Financial Self Sovereignty. Minimal technical knowledge is needed to setup. Just requires following step by step instructions. A fun weekend family project? I would hope so! Please share to those that want to take Bitcoin more seriously but need coached through setting up the infrastructure.

## Configuration

**Linux - TOR - Bitcoin Core - Fulcrum (Full Index Server) - Sparrow Wallet [Private Electrum]**

Cold Storage: Multi-Sig Wallet with at least two different hardware wallets is recommended with this configuration for long term Bitcoin storage.

Note: The following versions were used and verified functional/stable as of July 13, 2024. If issues arise with future versions please consider:
- Tor Browser = 13.5
- Bitcoin Core = 27.0
- Fulcrum = 1.11
- Sparrow = 1.9.1-1

Note: If working with future versions then obviously adjust the code below to reflect that. Also replace any "blabla" with your computer’s username.

## Hardware

1. Source Linux Computer. I recommend https://system76.com/desktops/meerkat [2TB+]
	- Launch Lite Keyboard and Mouse from System 76 are awesome add-ons, if needed.
2. Source Hardware Wallet/s for cold storage. I recommend https://oplanbtc.com/hardware
	- If multisig wallet desired then get at least two different types to reduce single source attack vectors.
3. After initial Linux setup, update operating system if updates are available.
4. Miscellaneous 
	- Go to settings/power/power saving options/automatic suspend... Toggle “when idle” to OFF so this computer will always stay active.
	- Click on the hamburger button (3 horizontal lines) in one of the file explorers next to the exit button and select “show hidden files”
	- Go to Home Folder and open .bashrc file. 
		- At end of the file add the following two lines: 
			- `# add /.local/bin to PATH`
			- `export PATH=”/.local/bin:$PATH”`
		- Save .bashrc and restart computer
		- Verify variable was added by opening terminal and typing `echo $PATH`

## Software

### Tor Browser

1. Download Tor Browser - https://www.torproject.org/download/ [tar.xz]
2. Download Signature [.asc]
3. GPG verify file (explained here https://support.torproject.org/tbb/how-to-verify-signature/)
	- Open Terminal
	- Add `gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org`
	- Then `gpg --output ./tor.keyring --export 0xEF6E286DDA85EA2A4BA7DE684E2C6E8793298290`
	- Verify tor.keyring exists in home folder
	- Now `gpgv --keyring ./tor.keyring ~/Downloads/tor-browser-linux-x86_64-13.5.tar.xz.asc ~/Downloads/tor-browser-linux-x86_64-13.5.tar.xz`
	- Should output “Good Signature from Tor Browser Developers”... Proceed to install, if so:
	- Note: mentions refreshing the key. Not needed at this point since good signature and future updates will be done within browser without need for GPG. Will add the command here though just in case it’s ever needed… `gpg --refresh-keys EF6E286DDA85EA2A4BA7DE684E2C6E8793298290`
4. Install
	- Right Click on tor’s tar.xz file in Downloads Folder
	- Select ‘Extract Here...’ then move folder to Home Folder (where desktop, documents, downloads, etc is nested), or just Extract to Home Folder
	- Click on tor-browser-linux-x86_64-13.5 Folder (wherever it’s nested)
	- Right click on tor-browser folder
	- Select ‘Open in Terminal’
	- Add `./start-tor-browser.desktop` to that terminal which will launch browser.
	- Setup config as you want (ie go through and block access to location, camera, etc, switch search to Start Page (.onion), etc) then press Connect. Should say connected after awhile.
	- Close out of Tor then go back to the terminal that ran it and type in `./start-tor-browser.desktop --register-app` to create a clickable launch in app list.
	- It’s now a clickable icon in Applications at the Dashboard. Go to that and right click Tor Browser and select ‘Pin to Dash’
	- Remove FireFox from Dash by right clicking and selecting ‘Remove from Favorites’
	- NOTE: You need to open the tor browser anytime you want to create a tor connection. Maybe there is a way to just start the tor daemon and run in background but not a huge deal if the browser is just left open and minimized.

### Bitcoin Core

1. Download Core. Go to https://bitcoincore.org/en/releases/, click on release you would like to download then click on first link (ie ...bin/bitcoin-core-XX.X). Save to Downloads Folder. NOTE: Downloading through TOR Browser will prompt you to save in their browser/Downloads folder by default which is confusing when you think it went to your home/Downloads folder. I always select home/Downloads location to keep things consistent.
2. Whichever version you go with, download the SHA256SUMS and SHA256SUMS.asc within the same release bin to your Downloads Folder.
3. Load up trusted builders/keys to verify through GPG
	- Go to https://bitcoin.org/en/full-node#linux-instructions
	- Click on Wladimir J. van der Laan’s releases key link to download the asc file to Download Folder
	- Right click the Download Folder and ‘Open in Terminal’
	- `gpg --import laanwj-releases.asc`
	- `gpg --list-keys`
	- `gpg --edit-key laanwj@gmail.com`
	- `fpr` …fingerprint should match what’s on website (...36C2E964). If so, proceed:
	- `trust` then `4`
	- `quit`
	- Go to https://github.com/bitcoin-core/guix.sigs/tree/main/builder-keys and pick out 3 or 4 others (ie fanquake.gpg, sjors.gpg, TheCharlaten.gpg, theStack.gpg, laanwj.gpg). For each one:
		- Click on link then Download Raw File and save at Download Folder
		- `gpg --import fanquake.gpg`
		- Do next import until all are imported
		- `gpg --list-keys`
		- `gpg --edit-key fanquake@gmail.com`
		- `trust` then `4`
		- `quit`
		- Do next trust until all are trusted
4. GPG verify file with `gpg --verify SHA256SUMS.asc`. This should list various signatures and some (ie 3 or 4) should be “good” per above keys that were trusted. This proves SHA256SUMS file is valid.
5. `sha256sum bitcoin-27.0-x86_64-linux-gnu.tar.gz` will output the value to compare to SHA256SUMS
6. Open SHA256SUMS and find bitcoin-27.0-x86_64-linux-gnu.tar.gz ...Does the value match the file’s checksum? If so, it’s a valid file. Proceed.
7. Go to Download Folder and extract bitcoin tar.gz file to Home (or move it there)
	- To make future “change directory” commands (cd) less awkward I move the bitcoin-27.0 folder out of the parent folder and place it directly in the home folder then delete the empty bitcoin-27.0-x86_64-linux-gnu folder
8. Click on that bitcoin-27.0 folder
9. Right click bin folder and ‘Open in Terminal’
10. `chmod +x *` to make files executable (or right click/properties each)
11. List with `ls`
12. `sudo apt install libxcb-xinerama0 libxcb-cursor0` to make the following work. Enter password.
13. Verify Tor Browser is open and connected.
14. Note to self: if you have backup of the blockchain (.bitcoin folder) then add to appropriate location before proceeding so you don’t have to download the whole thing again.
15. Run `./bitcoin-qt`
16. Press OK (which will install to default directory and not prune). **DO NOT ENABLE PRUNE!**
17. Setup
	- Setup up TOR route
		- Settings -> Options -> Network
		- Select “Connect through SOCKS5 Proxy”
		- Proxy IP = 127.0.0.1
		- Port = 9150 ...press OK (could also be 9050, but appears 9150 is tor browser default. If you are unable to connect switch this value and try again)
	- Modify Config file 
		- Settings -> Options -> Main
		- Open Configuration File, Continue:
		- Type `server=1` to switch on RPC server (needed for Fulcrum full index server)
		- Press Enter to go to next line
		- Type `walletbroadcast=0` to prevent node from rebroadcasting transactions without TOR
		- Press Enter to go to next line
		- Type `txindex=1` (needed for Fulcrum to function)
		- “Optional for best results”: Type `zmqpubhashblock=tcp://0.0.0.0:8433`
		- [![CoreConfigFile](https://github.com/BitcoinBonfire/DIY-Bitcoin/blob/main/CoreConfigFile.jpg)]
		- Press Save then Exit
	- EXIT Core then relaunch so new settings are active... `./bitcoin-qt`
	- Should now have a “P” next to BTC in bottom right which designates proxy is enabled... [![CoreStatus](https://github.com/BitcoinBonfire/DIY-Bitcoin/blob/main/CoreStatus.jpg)]
	- Verify connections with peers by right clicking the Peer icon to the right of the “P” and selecting “Show Peers tab”. After awhile if no peers are listed try switching 9150 port to 9050, exit, then relaunch.
18. Allow Core to run until it’s synced with blockchain (typically 1-4days with fast internet). While this is happening, you can move onto next steps...

### Fulcrum Server

1. Download https://github.com/cculianu/Fulcrum/releases [...x86_64-linux.tar.gz]
2. Also download ...shasum.txt and ...shasums.txt.asc
3. GPG Verify
	- Click on ...pubkeys/calinkey.txt link above the assets then download raw file to Downloads folder
	- Open a terminal
	- `cd Downloads`
	- `gpg --import calinkey.txt`
	- `gpg --edit-key calin.culianu@gmail.com`
	- `trust` then `4`
	- `quit`
	- `gpg --verify Fulcrum-1.11.0-shasums.txt.asc` ...If good signature then proceed.
	- `sha256sum Fulcrum-1.11.0-x86_64-linux.tar.gz`
	- Open Fulcrum-1.11.0-shasums.txt
	- Does the value match the file’s checksum? If so, it’s a valid file. Proceed.
4. `cd ..` to go to home directory
5. `mkdir fulcrum` to make a new folder called fulcrum
6. `mkdir fulcrum_db`
7. `cd Downloads`
8. `tar xvf Fulcrum-1.11.0-x86_64-linux.tar.gz` to unzip
9. `mv Fulcrum-1.11.0-x86_64-linux/* /home/blabla/fulcrum` to move contents to recently created folder
10. `cd ..`
11. `cd fulcrum`
12. `openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem` to generate keys for ssl (to communicate in encrypted manner)
13. Will ask questions, leave all blank, just press enter until you get back to prompt
14. `ls` …should now see cert.pem and key.pem listed in the fulcrum folder.
15. `mv fulcrum-example-config.conf fulcrum.conf` to rename file (ie mv moves or renames)
16. `ls` ...yes, it’s been renamed.
17. `nano fulcrum.conf` to edit in nano
	- Change “datadir = /path/to/a/dir” to `datadir = /home/blabla/fulcrum_db`. Note: Press enter at the "#" after this so “# Windows:..” turns blue and goes to next line
	- Put a `#` in front of “rpcuser = Bob_The_Banker” to make it inactive
	- Put a `#` in front of “rpcpassword = hunter1”
	- Delete the "#" in front of “rpccookie =…” to make it active and replace path with `/home/blabla/.bitcoin/.cookie`
	- Delete the "#" in front of “ssl = 0.0.0.0:50002”
	- Delete the "#" in front of “cert =…” and replace path with `/home/blabla/fulcrum/cert.pem`
	- Delete the "#" in front of “key =…” and replace path with `/home/blabla/fulcrum/key.pem`
	- Delete the "#" in front of “peering = true” and replace true with `false`
	- Delete the "#" in front of “tor_proxy = 9050…” and replace 9050 with `127.0.0.1:9150`
		- Press enter at "#" after this so “# e.g. localhost…” moves to next line and turns blue
	- Press Ctrl + X to save then press Y then Enter to exit and return to terminal
18. Make Fulcrum start on boot...
19. `sudo nano /etc/systemd/system/fulcrum.service` then enter password to create a service file then add the following to it:

> [Unit]   
> Description=Fulcrum   
> After=network.target   
>
> [Service]   
> ExecStart=/home/blabla/fulcrum/Fulcrum /home/blabla/fulcrum/fulcrum.conf   
> User=blabla  
> LimitNOFILE=8192   
> TimeoutStopSec=30min  
> 
> [Install]   
> WantedBy=multi-user.target  

20. Ctrl + X then Y then Enter to save/exit.
21. **STOP HERE IF YOUR BITCOIN CORE IS STILL SYNCING. ONCE SYNCED THEN PROCEED…**
22. `sudo systemctl enable fulcrum.service`
23. `sudo systemctl start fulcrum.service` (note: press up arrow to load previous commands... press up then delete “enable” and replace with “start” then press enter. Also if ctrl + v doesn’t paste try ctrl + shift + v in terminal but sometimes you need to delete some random variables before and the ~ after for some reason)
24. `sudo systemctl status fulcrum.service` ...should be running
25. Press `q` to quit
26. `journalctl -fu fulcrum.service` to monitor buildout of database. This will take some time. Eventually it will be downloaded and should say “...starting listening service for SslSrv 0.0.0.0:50002…” and “…listening for connections”
27. Optional: Check hard drive space 
	- Open new terminal
	- `df -h` …in general
	- `cd fulcrum_db`
	- `du -h` …fulcrum_db folder specific.

### Sparrow Wallet

1. Download Sparrow Wallet - https://sparrowwallet.com/download/ [...amd64.deb]
2. GPG verify file
	- Open a new terminal
	- `curl https://keybase.io/craigraw/pgp_keys.asc | gpg --import`
	- `gpg --edit-key craig@sparrowwallet.com`
	- `trust` then `4`
	- `quit`
	- `cd Downloads`
	- `gpg --verify sparrow-1.9.1-manifest.txt.asc` ...Should say “good signature from Craig Raw”. If so, proceed.
	- `sha256sum sparrow_1.9.1-1_amd64.deb`
	- Open sparrow-1.9.1-manifest.txt
	- Does the value match the file’s checksum? If so, it’s a valid file. Proceed.
3. Install Sparrow.
	- Go to Downloads folder and double click on sparrow_1.9.1-1_amd64.deb
	- Click Install then put in password.
4. **STOP HERE IF YOUR FULCRUM SERVER IS STILL SYNCING. ONCE SYNCED THEN PROCEED…**
5. Click on Show Application at Dash and click on Sparrow
6. Click Next until you can press Configure Server.
7. **SELECT PRIVATE ELECTRUM**
	- URL = `localhost` and `50002`
	- Toggle “Use SSL” ON
	- Toggle “Use Proxy” ON
	- Proxy url = `localhost` and `9150` (or possibly 9050 depending on default tor setting)
	- [![SparrowServerSettings](https://github.com/BitcoinBonfire/DIY-Bitcoin/blob/main/SparrowServerSettings.jpg)]
	- Press Test Connection… should say “connected to Fulcrum...”
8. Might be able to press Cancel here but I pressed “Create Wallet” called it “test” (knowing I was going to delete it later)
9. Either way I’m wanting to see the onion icon and blue toggle switch in bottom right corner... [![SparrowStatus](https://github.com/BitcoinBonfire/DIY-Bitcoin/blob/main/SparrowStatus.jpg)] ...should also say “connected to… 50002…” briefly on bottom when it loads). If so, success!
10. Optional layer to make wallet less accessible on boot: File -> Preferences -> General -> Wallet -> Load Recent Wallets ...Toggle OFF so opening Sparrow will not preload a wallet. You could put wallet file in something encrypted like Veracrypt or hidden on computer somewhere so you have to unlock/find file THEN load it.
11. **To get hardware wallets to work on Linux**
	- Within Sparrow go to TOOLS -> Install Udev Rules
	- Copy the sudo command
	- Open a new terminal
	- Paste (ctrl + shift + v) command. Again, might need to delete [[200 in front of sudo and ~ at end to clean up the command.
	- Press Enter then password … should say “success : true”
	- Note it didn’t work the first time. I closed out of Sparrow and reopened and repeated these steps with a newly displayed sudo command.
12. Load existing wallet (File -> Open Wallet) or Setup a new hardware wallet (File -> New Wallet)
13. Optional: Delete test wallet 
	- Open .sparrow folder at Home then wallets folder
	- Right click test file and select Move to Trash

## On-ramp / Off-ramp

Recommend https://river.com/ for converting fiat to bitcoin.
1. Links to bank account
2. Ability to setup recurring buy orders which deposit to River Account
3. Ability to setup automated withdrawals from River Account to Cold Storage
4. Nothing compares to River IMO.

## Load

1. Open Tor Browser to create a TOR Route.
2. Bitcoin Core:
	- Open Terminal
	- `cd bitcoin-27.0/bin`
	- `./bitcoin-qt`
	- This opens the core (note: this could be a button click like Tor Browser (--register-app), if desired. I like to keep a portion of this startup in terminal to get used to using it since I’m new to Linux world.
3. Fulcrum *starts automatically at boot* but `journalctl -fu fulcrum.service` to monitor fulcrum if needed
	- If you choose not to setup “start on boot” for Fulcrum then open as follows:
		- Right click Terminal
		- Select New Window
		- `cd fulcrum`
		- `Fulcrum /home/blabla/fulcrum/fulcrum.conf`
4. Sparrow
	- Click Show Applications at Dash
	- Click on Sparrow (could pin this to dash if you prefer)
	- Load Wallet

If you’ve made it this far, congratulations! Some might say “I found this useful/valuable and would like to return love/karma to the author.” Others might say “I would like to test this awesome new setup by sending some Bitcoin.” Either way, here is my (BitcoinBonfire) bitcoin address:

bc1qjlwmfgk77phhggquytkffsa3e8uckdavzpvslulepneqenknphksedr4qh

[![BitcoinBonfire](https://github.com/BitcoinBonfire/DIY-Bitcoin/blob/main/BitcoinBonfire.jpg)]

Much love! Keep stackin!

## Footnotes

1. I respect the Sparrow Wallet project, find it most user friendly, and agree with their website’s current stance at time of mapping out these instructions, July 13, 2024: “For this reason, Fulcrum emerges as a clear winner in this benchmark... Fulcrum is recommended as the ideal server to pair with Sparrow” found here https://sparrowwallet.com/docs/server-performance.html.
2. Note: As I mentioned, I'm a Linux noob. There is certainly a cleaner way to approach some of the steps such as converting the pointing/clicking steps with terminal commands. Decided to leave as is to show that the option to operate outside of the terminal is sometimes an option, if you prefer. My goal was to be able to give this to an 8 year old or 90 year old and have them set this up themselves.
3. The better approach here would be to record my screen and talk through this process and upload to Rumble and others so there is a visual to follow along with in case it’s needed. I, unfortunately, do not have the time to do this. Any volunteers? In fact, wouldn’t it be epic if someone recorded a little kid setting up this configuration step by step for effect (ie no excuses if a little kid can do it).
4. More info on GPG: https://www.gnupg.org/gph/en/manual/book1.html. 
5. One might ask “why are you referencing a Bonfire in the Dark Souls videogame?”
	- A Bonfire in real life is “a large and controlled outdoor fire, used either for informal disposal of burnable waste material or as part of a celebration.” I feel this is perfect analogy for Bitcoin... an informal disposal of the old system (fiat and other systems of control) considered “burnable waste” and we are all around this bonfire, we call bitcoin, celebrating the paradigm shift.
	- Bonfire Lore in Dark Souls: “When the fire fades the Undead Curse descends upon man and eternally links you to the bonfires. Thus, every time you die you spawn right back at one until the fire is linked. On top of this, the Dark Sign is slowly sapping away at your humanity leaving you closer and closer to humanity's true state, the Hollow. Without your humanity or souls you revert back into a Hollow husk just like in the time before the Fire was born. The bonfires seem directly connected to the First Flame since they have mysterious effects and powers. This entire system was setup by Gwyn to shepherd humanity to the fire and to keep humanity's dark power in check. All of this was done to extend his Age of Fire and the reign of the gods.”
	- All that to say Bitcoin = Bonfire on a few levels. I feel like this could generate a few potent memes. Maybe could induce some gamers to put down their controllers for a while, understand bitcoin, and setup a configuration like this.
