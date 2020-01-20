# Fixing Desktop ECs



# The easy way

Open SSDTTime on the machine in either Windows or Linux, and run the following:

* Dump DSDT
* Fake EC

After running both options, you'll be provided with `SSDT-EC.aml`. The .aml is super important as this is the compiled version, .dsl is decompiled and can be seen as source code.

Adding to Clover:

* EFI/CLOVER/API/patched

Adding to OpenCore:

* EFI/OC/ACPI
* config.plist -> ACPI -> Add


# The long way

Having issues with SSDTTime or wanna learn the process? Well continue on!

## Getting a copy of your DSDT

So to get a copy of your DSDT there's a couple of options:

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
  * For OpenCore only, add this to `EFI/OC/Tools` and in your config under `Misc -> Tools` with the argument: `-b -n DSDT -z` and select this option in OpenCore's picker. Rename the DSDT.dat to DSDT.aml. Tool is provided by [acpica](https://github.com/acpica/acpica/tree/master/source/tools/acpidump)

If OpenCore is having issues running acpidump, you can call it from the shell with [OpenCoreShell](https://github.com/acidanthera/OpenCoreShell/releases)\(reminder to add to both `EFI/OC/Tools` and in your config under `Misc -> Tools` \):

```text
shell> fs0: // replace with proper drive

fs0:\> dir // to verify this is the right directory

Directory of fs0:\

01/01/01 3:30p EFI

fs0:\> cd EFI\OC\Tools // note that it's with forward slashes

fs0:\EFI\OC\Tools> acpidump.efi -b -n DSDT -z
```

## Decompiling the DSDT

So once we have our DSDT we still got a bit of work left, we'll first want to decompile it so we can view it easier. Couple options:

### macOS

So decompiling DSDTs is quite easy with macOS, all you need is [MaciASL](https://github.com/acidanthera/MaciASL). To decompile, just open the file!

### Windows

Decompiling on windows is fairly simple as well, you will need [iasl.exe](https://acpica.org/sites/acpica/files/iasl-win-20180105.zip) and Command Prompt:

```text
path/to/iasl.exe path/to/DSDT.aml
```

![](https://cdn.discordapp.com/attachments/456913818467958789/668211002340409404/unknown.png)

### Linux

Compiling and decompiling with Linux is just as simple, you will need a special copy of [iasl](http://amdosx.kellynet.nl/iasl.zip) and terminal:

```text
path/to/iasl path/to/DSDT.aml
```

## Creating SSDTs

This one's fairly easy to figure out, open your decompiled DSDT and search for `PNP0C09`. This should give you a result like this:

![](https://i.imgur.com/lQ4kpb9.png)

As you can see our `PNP0C09` is found within the `Device (EC0)` meaning this is the device we want to hide from macOS(others may find ). Now grab our SSDT-EC and uncomment the EC0 function(remove the `/*` and `*/` around it):

* [SSDT-EC-USBX](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC-USBX.dsl)
  * For Skylake+ and all AMD systems
* [SSDT-EC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC.dsl)
  * For Haswell and older

```text
/* <- REMOVE THIS
External (_SB_.PCI0.LPCB.EC0, DeviceObj)

   Scope (\_SB.PCI0.LPCB.EC0)
   {   
      Method (_STA, 0, NotSerialized) // _STA: Status
      {
         If (_OSI ("Darwin"))
         {
            Return (0)
         }
         Else
      {
      Return (0x0F)
     }
  }
}
*/ <- REMOVE THIS
```

But looking back at the screenshot above we notice something, our ACPI path is different: `PC00.LPC0` vs `PCI0.LPCB`. This is very important especially when you're dealing with Intel consumer vs Intel HEDT and AMD, `PC00.LPC0` is common on Intel HEDT while `PCI0.SBRG` is common on AMD. And they even come with name variation such as  `EC0`, `H_EC`, `PGEC` and `ECDV`, so there can't be a one size fits all SSDT, **always verify your path and device**.

And make sure to scroll to the bottom as the new Fake EC function also need the correct path to replace the old EC.

> What happens if multiple `PNP0C09` show up

When this happens you need to figure out which is the main and which is not, it's fairly easy to figure out. Check each controller for the following properties:

* `_HID`
* `_CRS`
* `_GPE`

> What happens if no `PNP0C09` show up?

This means your SSDT can be *almost* complied, the main thing to watch for is whether your DSDT uses `PCI0.LPCB` or not. The reason being is that we have a FakeEC at the bottom of our SSDT that needs to connect properly into our DSDT. Gernally AMD uses `SBRG` while Intel HEDT use `LPC0`, **verify which show up in your DSDT**. Once you find out, change `PCI0.LPCB` to your correct path:

```text
Scope (\_SB.PCI0.LPCB)
{
    Device (EC)
    {
        Name (_HID, "ACID0001")  // _HID: Hardware ID
        Method (_STA, 0, NotSerialized)  // _STA: Status
        {
            If (_OSI ("Darwin"))
            {
                Return (0x0F)
            }
            Else
            {
                Return (Zero)
            }
        }
    }
}
```


> Hey what about USBX? Do I need to do anything?

USBX is universal across all systems, it just creates a USBX device that forces USB power properties. This is crucial for fixing Mics, DACs, Webcams, Bluetooth Dongles and other high power draw devices. This is not mandatory to boot but should be added in post-install if not before. Note that USBX is only used on skylake+ systems, Broadwell and older can ignore and that USBX requires a patched EC to function correctly


## Adding your SSDT

Adding to Clover:

* EFI/CLOVER/API/patched

Adding to OpenCore:

* EFI/OC/ACPI
* config.plist -> ACPI -> Add
