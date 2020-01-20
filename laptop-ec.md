# Fixing Laptop ECs

To start, we're going to need a copy of your DSDT, to dump it from your firmware you got a couple options:

* [MaciASL](https://github.com/acidanthera/MaciASL/releases)
  * Open the app on the target machine and the system's DSDT will show, then File -&gt; SaveAs `System DSDT`. Make sure the file format is ACPI Machine Language Binary\(.AML\), this will require the machine to be running macOS
  * Do note that all ACPI patches from clover/OpenCore will be applied to the DSDT
* [SSDTTime](https://github.com/corpnewt/SSDTTime)
  * Supports both Windows and Linux for DSDT dumping
* [acpidump.exe](https://acpica.org/sites/acpica/files/iasl-win-20180105.zip)
  * In command prompt run `path/to/acpidump.exe -b -n DSDT -z`, this will dump your DSDT as a .dat file. Rename this to DSDT.aml
* F4 in Clover Boot menu
  * DSDT can be found in `EFI/CLOVER/ACPI/origin`, the folder **must** exist before dumping
* [`acpidump.efi`](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/tree/master/extra-files/acpidump.efi.zip)
  * For Opencore only, add this to `EFI/OC/Tools` and in your config under `Misc -> Tools` with the argument: `-b -n DSDT -z` and select this option in OpenCore's picker. Rename the DSDT.dat to DSDT.aml. Tool is provided by [acpica](https://github.com/acpica/acpica/tree/master/source/tools/acpidump)

If OpenCore is having issues running acpidump, you can call it from the shell with [OpenCoreShell](https://github.com/acidanthera/OpenCoreShell/releases)\(reminder to add to both `EFI/OC/Tools` and in your config under `Misc -> Tools` \):

```text
shell> fs0: // replace with proper drive

fs0:\> dir // to verify this is the right directory

Directory of fs0:\

01/01/01 3:30p EFI

fs0:\> cd EFI\OC\Tools // note that it's with forward slashes

fs0:\EFI\OC\Tools> acpidump.efi -b -n DSDT -z
```

# Decompiling the DSDT

So once we have our DSDT we still got a bit of work left, we'll first want to decompile it so we can view it easier. Couple options:

## macOS

So decompiling DSDTs is quite easy with macOS, all you need is [MaciASL](https://github.com/acidanthera/MaciASL). To decompile, just open the file!

## Windows

Decompiling on windows is fairly simple as well, you will need [iasl.exe](https://acpica.org/sites/acpica/files/iasl-win-20180105.zip) and Command Prompt:

```text
path/to/iasl.exe path/to/DSDT.aml
```

![](https://cdn.discordapp.com/attachments/456913818467958789/668211002340409404/unknown.png)

## Linux

Compiling and decompiling with Linux is just as simple, you will need a special copy of [iasl](http://amdosx.kellynet.nl/iasl.zip) and terminal:

```text
path/to/iasl path/to/DSDT.aml
```

# Finding the right EC patch

Now that our DSDT is readable, next search for `PNP0C09`. Should give you something similar to this:

![](https://i.imgur.com/lQ4kpb9.png)

As you can see our `PNP0C09` is found within the `Device (EC0)` meaning this is the device we want to rename.

> What happens if multiple `PNP0C09` show up

When this happens you need to figure out which is the main and which is not, it's fairly easy to figure out. Check each controller for the following properties:

* `_HID`
* `_CRS`
* `_GPE`

Note that only the main EC needs renaming, if you only have one `PNP0C09` then it is automatically your main regardless of properties.


# Applying your EC patch

As you can see from the table below, we'll be renaming our EC listed in the DSDT. Do note you cannot just throw random renames without checking first, as this can cause actual damage to your laptop.

|Comment|Find\*\[HEX\]|Replace\[HEX\]|
|:-|:-|:-|
|change EC0 to EC|4543305f|45435f5f|
|change H\_EC to EC|485f4543|45435f5f|
|change ECDV to EC|45434456|45435f5f|
|change PGEC to EC|50474543|45435f5f|


## Clover users:

| Comment | String | Change XXXX to EC |
| :--- | :--- | :--- |
| Disabled | Boolean | No |
| Find | Data | xxxxxxxx |
| Replace | Data | xxxxxxxx |

![](https://cdn.discordapp.com/attachments/302485086060937219/668662065665409024/Screen_Shot_2020-01-19_at_8.44.00_PM.png)

![](https://cdn.discordapp.com/attachments/456913818467958789/668666485463318558/Screen_Shot_2020-01-19_at_9.01.20_PM.png)

## Opencore users:

| Comment | String | Change XXXX to EC |
| :--- | :--- | :--- |
| Enabled | String | YES |
| Count | Number | 0 |
| Limit | Nuber | 0 |
| Find | Data | xxxxxxxx |
| Replace | Data | xxxxxxxx |

![](https://cdn.discordapp.com/attachments/456913818467958789/668667268254793728/Screen_Shot_2020-01-19_at_9.04.50_PM.png)

