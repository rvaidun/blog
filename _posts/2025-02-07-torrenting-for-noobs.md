---
layout: post
title:  "Safely torrenting for noobs"
date:   2025-02-07 12:52:08 -0800
categories: general
---

This article will discuss how to safely torrent files. It is intended for people who are new to torrenting and want to learn how to do it safely.

# What is torrenting?
Torrenting is a way to share files over the internet. It works by breaking up the file into small pieces and distributing them across a network of computers. When you download a file using a torrent, you are actually downloading pieces of the file from multiple sources at the same time. This makes the download faster and more reliable than downloading the file from a single source. It also makes it harder for authorities to track who is hosting the file since the file is spread out across multiple servers/computers and taking down one server/computer won't stop the file from being shared. It is important to note that torrenting is not illegal, but downloading copyrighted material without permission is illegal. The Verge made a nice [video](https://www.youtube.com/watch?v=OFswNCU5CKA&ab_channel=TheVerge) explaining how torrenting works.



## Important things to keep in mind
 Here are some tips to keep in mind when torrenting:
- Always download torrents from trusted sources. Downloading torrents from untrusted sources can lead to malware being installed on your computer.
- Always use a VPN when torrenting. A VPN will encrypt your internet connection and hide your IP address, making it harder for people to track what you are downloading. Note anyone can see what IP addresses are downloading a torrent since it is a peer-to-peer network and your IP address is publicly visible, so if you do download copyrighted material without permission, don't be surprised if you get a letter from your ISP.

# Torrent Client
The first thing you need to do to start torrenting is to download a torrent client. A torrent client is a program that allows you to download and upload files using the BitTorrent protocol. There are many different torrent clients available, but I feel like most of them are bloated with ads and unnecessary features. This article will use the [transmission](https://transmissionbt.com/) client and how to use it through the command line. If you are scared of CLI and want a GUI transmission client, you can download it from [here](https://transmissionbt.com/download/).

# Installation
To install transmission on Ubuntu, you can run the following command:
```bash
sudo apt-get install transmission-cli
```
On macOS, you can install transmission using brew:
```bash
brew install transmission-cli
```

# Usage
For this example I will be downloading the latest Ubuntu ISO. To download the Ubuntu ISO, you first need to get a magnet link or a torrent file. You can get the torrent file from the [Ubuntu website](https://ubuntu.com/download/alternative-downloads). Once you have the torrent file, you can start the download by running the following command:
```bash
(venv) ➜  my-blog git:(main) ✗ transmission-daemon
(venv) ➜  my-blog git:(main) ✗ transmission-remote -a "/Users/rahulvaidun/Downloads/ubuntu-24.04.1-desktop-amd64.iso.torrent"
localhost:9091/transmission/rpc/ responded: success
```
The first command will start the transmission daemon if it is not started and only needs to be run once. The second command will add the torrent file to the transmission daemon and start the download. Please note that you can also use magnet link after `-a` instead of the torrent file.

You can check the status of the download by running the following command:
```bash
(venv) ➜  my-blog git:(main) ✗ transmission-remote -l
    ID   Done       Have  ETA           Up    Down  Ratio  Status       Name
    15     2%   106.8 MB  15 min       0.0  6463.0   0.00  Downloading  ubuntu-24.04.1-desktop-amd64.iso
Sum:            106.8 MB               0.0  6463.0
```
Here you can see the status of the download, how much of the file has been downloaded, the estimated time remaining, the upload and download speed, and the name of the file being downloaded.
Here are some useful commands that you can use with transmission:
- `transmission-remote -a <torrent-file or magnet file>` - Add a torrent file to the transmission daemon.
- `transmission-remote -l` - List all the torrents being downloaded.
- `transmission-remote -t <torrent-id> -r` - Remove the torrent with the given id.
- `transmission-remote -t all --remove` - Remove all the torrents.
- `transmission-remote -t <torrent-id> -S` - Stop the torrent with the given id.
- `transmission-remote -t all --stop` - Stop all the torrents.
- `transmission-remote -t <torrent-id> -s` - Start the torrent with the given id.
- `transmission-remote -t all --start` - Start all the torrents.
- `transmission-remote -w ~/transmission_torrents` - Change the download location to the Downloads folder.
- `watch --interval 1 'transmission-remote -l'` - Watch the status of the torrents being downloaded every second.

Once the download is complete you can find the file in the Downloads folder. Please note usually once the download is complete, the torrent client will continue to seed the file. Seeding is when you continue to share the file with others who are downloading it. This is important because it helps to keep the file available for others to download. If you don't want to seed the file, you can remove the torrent from the client with the `transmission-remote -t <torrent-id> -r` command. Also I would probably alias the `transmission-remote` command to something shorter like `tr` to make it easier to type.

# Conclusion

I hope this article helps you get started with torrenting.


