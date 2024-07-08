# Downloading, building, and running the node from source code
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Introduction](#introduction)
- [Step 1: Install Erlang](#step-1-install-erlang)
  - [Install Erlang using the package manager](#install-erlang-using-the-package-manager)
  - [Install Erlang using `kerl`](#install-erlang-using-kerl)
- [Step 2: Download and build the node](#step-2-download-and-build-the-node)
- [Step 3: Create directories and place the files](#step-3-create-directories-and-place-the-files)
- [Step 4: Start the node](#step-4-start-the-node)
- [How to stop the node?](#how-to-stop-the-node)
- [Step 5: Get the certificate](#step-5-get-the-certificate)
- [How to check, if my node works?](#how-to-check-if-my-node-works)
- [What do I need to do if something goes wrong?](#what-do-i-need-to-do-if-something-goes-wrong)
  - [Troubleshooting](#troubleshooting)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction

If you want to go through the advanced way of building sources, this manual will help you do this.

## Step 1: Install Erlang

You can install Erlang either using the package manager, or using `kerl`.

### Install Erlang using the package manager

If you use the recommended Ubuntu version (22.04.1), you can install Erlang using the package manager. To do this, run the following command:

   ```bash
   apt install cmake clang gcc git curl libssl-dev build-essential automake autoconf libncurses5-dev elixir erlang
   ```

:::note

If you use another version of Ubuntu (20.04, for instance), refer to the section below to install Erlang using `kerl`.

:::

### Install Erlang using `kerl`

To install erlang using `kerl`:

1. Install the software you need to seamlessly install Erlang:

   ```bash
   apt install cmake clang gcc git curl libssl-dev build-essential automake autoconf libncurses5-dev elixir erlang-base erlang-public-key erlang-asn1 erlang-ssl erlang-dev erlang-inets erlang-eunit
   ```

2. Create a directory, where Erlang will be installed to:

   ```bash
   mkdir /opt/erlang
   ```

3. Create a directory, where you will work with your node:

   ```bash
   mkdir /opt/<your_node>
   ```

4. Go to `/opt/erlang` directory:

   ```bash
   cd /opt/erlang
   ```

5. Install Erlang. To do this, download the `kerl` script:

   ```bash
   curl -O https://raw.githubusercontent.com/kerl/kerl/master/kerl
   ```

   > **Hint**
   >
   > If you already have Erlang installed on your machine, we strongly recommend deleting it before the new installation, using the following command:
   >
   > ```bash
   > apt purge erlang*
   > ```

6. Change script mode to executable by using the following command:

   ```bash
   chmod a+x kerl
   ```

7. Update the list of Erlang releases using the following command:

    ```bash
    ./kerl update releases
    ```

8. Build the release 24.3 using the following command:

   ```bash
   ./kerl build 24.3
   ```

   After installation is complete, you will see the following message in the console:

    ```text
    Erlang/OTP 24.3 (24.3) has been successfully built
    ```
9. Create a new subdirectory in `erlang`:

   ```bash
   mkdir /opt/erlang/<your_directory_name>
   ```
10. Install Erlang to the subdirectory you've created on step 10 using the following command:

    ```bash
    ./kerl install 24.3 /opt/erlang/<your_directory_name>
    ```

11. Run the following command to activate the Erlang installation:

    ```bash
    source /opt/erlang/<your_directory_name>/activate
    ```

:::note

 If your Erlang installation is not activated, you will NOT be able to run the Tea Ceremony and build the node.

:::

## Step 2: Download and build the node

:::note

Choose a project folder to clone your project into. Use this folder to build the node.

:::

1. Download the node sources from Github into your working directory (`your_node`, for instance), using the following command:

    ```bash
    git clone https://github.com/thepower/tpnode.git -b e24
    ```

2. Go to `tpnode` directory, using the command:

   ```bash
   cd tpnode
   ```

3. Check and download the dependencies using the command:

   ```bash
   ./rebar3 get-deps
   ```

4. Compile the node source by running the following command:

   ```bash
   ./rebar3 compile
   ```

5. Build the release by running the following command:

   ```bash
   ./rebar3 release
   ```

6. Copy the node directory from `_build/default/rel/thepower` to `/opt` by running the following command:

   ```bash
   cp -r _build/default/rel/thepower /opt
   ```

## Step 3: Create directories and place the files

> **Hint**
>
> You can create an additional directory named `thepower`, for example, and place `db` and `log` as subdirectories there.

To create directories for files:

1. Go to your working directory using the following command:

   ```bash
   cd /opt/thepower
   ```

2. Create `db` and `log` directories in your working directory (`/opt/thepower`, for instance) using the following command:

   ```bash
   mkdir {db,log}
   ```

3. Place `genesis.txt` and `node.config` near these directories using the following commands:

   ```bash
   cp ~/tea_ceremony_directory/node.config /opt/thepower/node.config
   ```

   ```bash
   cp ~/tea_ceremony_directory/genesis.txt /opt/thepower/genesis.txt
   ```

:::warning

The private key you get with the `node.config` file cannot be restored, if you lose it. Please, store it securely: place a backup onto your local machine.

:::

## Step 4: Start the node

Start the node using `systemd`. To do this:

1. Create a file `tpnode.service` under `/etc/systemd/system` directory.
2. The file must contain the following:

   ```bash
   [Unit]
   Description=tpnode service
   Requires=network.target
   After=network.target

   [Service]
   Type=forking
   ExecStart=/opt/thepower/bin/thepower start
   ExecStop=/opt/thepower/bin/thepower stop
   User=root
   Group=root
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

3. Run the following command to rerun all generators , reload all unit files, and recreate the entire dependency tree. While the daemon is being reloaded, all sockets systemd listens on behalf of user configuration will stay accessible:

   ```bash
   systemctl daemon-reload
   ```

4. Run the following command to enable the service after reboot:

   ```bash
   systemctl enable tpnode.service
   ```

5. Start the node using the following command:

   ```bash
   systemctl start tpnode.service
   ```

## How to stop the node?

To stop the node, run:

```bash
systemctl stop tpnode.service
```

## Step 5: Get the certificate

[Obtain the SSL certificate for your node](https://doc.thepower.io/docs/Maintain/build-and-start-a-node/ssl-certs-for-node) and place it into the `db` directory.

## How to check, if my node works?

To check, if your node works, run:

```bash
curl http://your_node.example.com:1080/api/node/status | jq
```

where:

- `your_node.example.com` — your node address;
- `1080` — port, that your node uses for `api`.

Replace the example parameters with the ones you need.

## What do I need to do if something goes wrong?

:::caution

If something goes wrong, go to the `log` folder, and read the logs. If there are errors, write to Power Ecosystem Telegram chat: `https://t.me/thepower_chat`.

:::

### Troubleshooting

1. You get the following error:

    ```bash
    ===> Failed to boot tpnode for reason {{{badmatch,undefined},
    [{nodekey,get_priv,0,
    [{file,
    "/home/thepower/tpnode/apps/tpnode/src/nodekey.erl"},
    {line,26}]},
    {tpic2,certificate,0,
    [{file,
    "/home/thepower/tpnode/apps/tpic2/src/tpic2.erl"},
    {line,196}]},
    {tpic2,
    '-childspec/0-fun-0-',2,
    [{file,
    "/home/thepower/tpnode/apps/tpic2/src/tpic2.erl"},
    {line,217}]},
    {tpic2,childspec,0,
    [{file,
    "/home/thepower/tpnode/apps/tpic2/src/tpic2.erl"},
    {line,229}]},
    {tpnode_sup,init,1,
    [{file,
    "/home/thepower/tpnode/apps/tpnode/src/tpnode_sup.erl"},
    {line,88}]},
    {supervisor,init,1,
    [{file,"supervisor.erl"},
    {line,295}]},
    {gen_server,init_it,2,
    [{file,"gen_server.erl"},
    {line,374}]},
    {gen_server,init_it,6,
    [{file,"gen_server.erl"},
    {line,342}]}]},
    {tpnode,start,[normal,[]]}}
    ```
   **Reason**

   You don't have `genesis.txt` and `node.config` files.

   **Solution**

   Start the Tea Ceremony client **BEFORE** starting the node. It will get proper `genesis.txt` and `node.config`.

2. You get the following error when starting the Tea Ceremony client:

   ```bash
    ~/tpnode# ./teaclient DEE570BD76F3
    -bash: ./teaclient: Permission denied
   ```

   **Reason**

   Probably, you haven't changed `teaclient` file mode to executable.

   **Solution**

   Change `teaclient` file mode to executable by running the following command:

   ```bash
   chmod +x tea*
   ```
3. You get the following error when starting the Tea Ceremony client:

   ```bash
   ~# ./teaclient -n demonode05 DEE570BD76F3
   ceremony client connecting to tea.thepower.io:443
   Server rejects connection, reason: bad_token
   ```

   **Reason**

   Token `DEE570BD76F3` has been used by another user.

   **Solution**

   Get the new token from the Telegram bot.

4. You get the following error when starting the Tea Ceremony client:

   ```bash
   ceremony client connecting to knuth.cleverfox.ru:1436
   =WARNING REPORT==== 19-Oct-2022::14:32:54.056133 ===
   Description: "Authenticity is not established by certificate path validation"
   Reason: "Option {verify, verify_peer} and cacertfile/cacerts is missing"

   Server rejects connection, reason: you_are_late
   ```

   **Reason**

   You are late for the tea ceremony.

   **Solution**

   Stick to the bot recommendations and don't be late.

5. You see different ports displayed in server control panel and written in firewall rules.

   **Reason**

   Problems with ports.

   **Solution**

   Check your `iptables` rules:

   ```bash
   iptables -L -nv
   ```

   Find the rules for all three ports among the entries:

   1. If the rules are absent, please contact or tech support in our [Telegram chat](https://t.me/thepower_chat).
   2. If you've found a corrupted port, try to recreate rule using the server control panel.

   :::warning

   **The Power Ecosystem is not responsible for the result of actions described below!** These actions can lead to loss of access to the server.

   :::

   Another way of addressing the problem is to:

   1. Save the rules into a file using the following command:

      ```bash
      iptables-save -f iptables.save
      ```
   2. Edit the corrupted port:

      ```bash
      vim iptables.save
      ```

   3. Restore the rules from the file:

      ```bash
      iptables-restore iptables.save
      ```