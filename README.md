# Fixing ECs for macOS Catalina

So you've come here for 1 of 2 reasons, you've either been told macOS catalina requires the embedded controller to be corrected or you hit a roadblock in your install like one of these:
```text
apfs_module_start...
```

```text
Waiting for Root device
```

```text
Waiting on...IOResources...
```

```text
previous shutdown cause...
```
## So why do I get these errors?

So with macOS catalina, there were some changes in how AppleACPIEC works which makes it so when it doesn't pass the checks it'll therefore stall. Specifically the Embedded Controller(EC) has new processes happen to it:

1. AppleACPIPlatform.kext loads and sets all devices with the ACPI name of EC__ and device PNP0C09 the property of boot-ec
2. It then hands off control to its plugin, AppleACPIEC.kext, and starts a probe for either PNP0C09 or boot-ec
3. When loaded, it will then verify for the other meaning we must have both `PNP0C09` and `boot-ec`. If not, macOS will just get stuck but due to the nature of parallel kext loading we don't explicitly see the error instead seeing errors such as `apfs_module_start...`, `Waiting for Root device`, `Waiting on...IOResources...`, `previous shutdown cause...`, etc. And guess what, most PCs don't have their embedded controller named `EC__` instead known by `EC0_`, `H_EC`, `ECDV` and `PGEC`.(Some Lenovos are the rare exception, though not all models)


## So how do I fix this? 

Well depending on your machine, there are different fixes. See below for your specific system:

* [Desktop and Laptop users](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix.html)
* [Samsung smart oven users](https://www.youtube.com/watch?v=dQw4w9WgXcQ)

