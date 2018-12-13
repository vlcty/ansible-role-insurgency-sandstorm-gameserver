# ansible-role-insurgency-sandstorm

Deploy Insurgency Sandstorm gameservers on Linux with ansible!

## Variables

### Global variables

See `defaults/main.yml` for a commented list of predefined variables.

### Example config

Here is an complete example config in my host_vars folder for my gameserver:

```
sandstorm_servers:
    coop:
        sandstorm_servername: More or less hardcore coop # mandatory
        sandstorm_server_start_map: Farmhouse # mandatory
        sandstorm_start_scenario: Scenario_Farmhouse_Checkpoint_Security # mandatory
        sandstorm_port: 27333 # mandatory
        sandstorm_admins: # optional
            - 76561198003172459 # veloc1ty
        sandstorm_bans: # optional
            - 76561197987793486
        sandstorm_mapcycle: # optional
            - Scenario_Crossing_Checkpoint_Security
            - Scenario_Hideout_Checkpoint_Security
            - Scenario_Precinct_Checkpoint_Security
            - Scenario_Refinery_Checkpoint_Security
            - Scenario_Farmhouse_Checkpoint_Security
            - Scenario_Summit_Checkpoint_Security
        sandstorm_config: | # mandatory
            [/script/insurgency/insurgency.insgamemode]
            ObjectiveSpeedup=0.5

            [/script/insurgency.insmultiplayermode]
            InitialSupply=25
            PostRoundTime=3
            FriendlyFireModifier=1.0
            FriendlyFireReflect=0.9

            [/script/insurgency.inscoopmode]
            bUseVehicleInsertion=False
            MinimumEnemies=6
            MaximumEnemies=20
```

Every key in the dictionary `sandstorm_servers` is a gameserver. Add more keys with different names to spawn more. In my example only the gameserver "coop" will be created. I currently only have one.

The config for each server should be self explaining. Most of the variables are mandatory.

## Tool: steamid.io

Admin Steam IDs have to be in the SteamID64 format. You can convert a profile URL to SteamID64 using this online tool: https://steamid.io/

## Note: 32bit libs

On 64 bit systems some 32 bit libs have to be installed. Insurgency Sandstorm is a 64bit executable, but steamcmd is not. Blame Steam for that.

On Archlinux you have to activat the `multilib` repo. Open `/etc/pacman.conf` and uncomment the following block:

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Then install the 32 bit gcc libs:

```
pacman -Syu lib32-gcc-libs
```

Documentation: https://wiki.archlinux.org/index.php/official_repositories#Enabling_multilib

You can also do this with ansible:

```
- name: Enabling multilib
  blockinfile:
    path: /etc/pacman.conf
    block: |
        [multilib]
        Include = /etc/pacman.d/mirrorlist
    state: present

- name: Install lib32-gcc-libs
  pacman:
     name: lib32-gcc-libs
     state: present
     update_cache: true
```

## Supported distributions

I've developed this role against Archlinux. All other distributions should also work.

## Developer Documentation

### setuid bit on steamcmd

I set the setuid bit on the steamcmd binary to run with the user rights. This is because changing running user in ansible is not very good looking. The setuid bit helps here.

### Why not calling the gameserver binary directly in the systemd service

My first approach was to call the binary directly in the `ExecStart` key in the systemd service file. But systemd didn't want to load the file because of the ? character in the binary name. Escaping it didn't help. So I had to go the way around it using a startscript.

## Further reading

Read this google document: https://docs.google.com/document/d/1GDLg5p9jjeIya7EgBk0ibzDtDlyQ-U_jpspOzby-JmM/edit

or my blogpost: https://blog.veloc1ty.de/2018/12/09/starting-a-dedicated-insurgency-sandstorm-server-on-linux/
