# Check SSD Life

This script measures a specific SMART attribute of an SSD in order to determine its **"life used"**.

It is written in bash, and should work for all NVMe SSD's. For ATA SSD's, it won't work for every brand and model, but it is easily customizable as long as you know what SMART attribute to use and which type of value to measure.

It mainly relies on [smartmontools'](https://www.smartmontools.org/) `smartctl` utility, but it can also incorporate custom SMART attribute measuring tools like [Innodisk's iSMART](https://www.innodisk.com/en/software/solutions/ismart).

### Sample usage

```
$ check_ssd_life -d /dev/sda
OK: SSD life used is at 55%

$ check_ssd_life -d /dev/nvme0 -w 50
WARNING: SSD life used is at 55%

$ check_ssd_life -w 50% -c 80% -t /path/to/iSMART_64 -d /dev/sdb
CRITICAL: SSD life used is at 85%
```

### Exit codes and states
0: OK<br>
1: WARNING<br>
2: CRITICAL<br>
3: UNKNOWN

### Default thresholds
WARNING: 80%<br>
CRITICAL: 90%

### Notable dependencies
- `bash` 4.0 or later
- `bc`
- `smartmontools`

### Adding ATA SSD models

If you are adding a new ATA SSD model to this script, follow these instructions:

1. First figure out what tool to use to get SMART data from the model of ATA SSD's. You'll most likely want to use `smartctl`, but sometimes `smartctl` just won't do, such as with Innodisk ATA SSD's. If `smartctl` won't do, you'll need to find a custom tool that you can use for that particular ATA SSD model, like how we use Innodisk's iSMART tool for their ATA SSD's.

2. Once you figure out what tool to use, figure out what SMART attribute to choose as a measure of ATA SSD life

3. Once you have chosen a SMART attribute, select whether to use that attribute's worst value, raw value, or normalized value. You might not need to do this if you're creating a custom check.

4. Once you know all of that, add them to their respective arrays: `models`, `identifier`, `attribute`, and `value`. If you're using `smartctl` for the model, you're done! If a custom tool is needed, you'll have to make a custom check and add it to this script in the code section between "**CUSTOM LIFE USED CHECKS START HERE**" and "**CUSTOM LIFE USED CHECKS END HERE**", along with installing the custom tool on the host machine.

### Currently supported custom checks and custom tools

#### Innodisk ATA SSD's with Innodisk iSMART

You will need to obtain the Innodisk iSMART tool by making an inquiry on [Innodisk's website](https://www.innodisk.com/en/inquiry). Then, install iSMART on the host machine and use the `-t` option to tell the script where to find it.

In my testing, placing only the `iSMART_64` binary in the host, making it executable, and using `-t /path/to/iSMART_64` is sufficient. Making sure that only root owns it and can execute it is also recommended.

There are minor, seemingly insignificant errors when not including the configuration files that are supposed to come with it. If running just the `iSMART_64` binary by itself does not show the "Health" attribute, and there are errors shown because of the missing config files, try adding them into the same directory that the `iSMART_64` binary is in and see if that gets the "Health" attribute to show up.

### Comparison to alternatives
Thomas Krenn's [check_smart_attributes](https://github.com/thomas-krenn/check_smart_attributes) tool is a more comprehensive tool that performs a similar function. Being more comprehensive, however, means it also has functions that aren't related to measuring an SSD's lifespan. check_ssd_life was created for the sole purpose of measuring an SSD's life used, and standardizing ATA SSD measurements _"the NVMe way"_ to make the data easier to organize, along with lots of error checking for when something (inevitably) goes wrong.
