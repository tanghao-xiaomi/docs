# xTS Certification Test Case Essential Collection

[ English | [简体中文](../../zh-cn/test_dev_guide/openvela_xts_test_cases.md) ]

## Document Description

This open source document is intended to provide individual developers with a standardized reference framework for self-test cases, helping users quickly understand the case composition, test methods, and adaptation rules. By clarifying the test scope and process, it improves test efficiency and the quality of compatibility verification.


1. This document is used by individual developers to quickly understand the composition of self-test cases and test methods, and to select adaptation test cases based on chip characteristics to improve test efficiency.
2. The General Self-Test Cases and Category-Specific Self-Test Cases in the test cases are test types. The General Self-Test Cases are mandatory, while the Category-Specific Self-Test Cases need to be selectively tested based on product characteristics.
3. If a test case is not applicable or you have questions about the test case steps, you can describe the main issue in the open source forum, and the community maintenance team will review and provide feedback periodically.
4. Test cases of the "Functional" test type can directly use an ordinary office environment or home environment, with no mandatory requirements for environmental details. For test cases of the "Performance" or "Stability" test type, testing must be conducted in a clean, interference-free environment (such as Bluetooth).

| Department                          | Latest Version | Effective Date |
| ----------------------------------- | -------------- | -------------- |
| openvela Community Maintenance Team | V1.0           | 2026-05-19     |

---

## I. General Self-Test Cases

### 1. Functional Testing

#### 1.1 System Kernel

##### 1.1.1 System Memory Management Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Memory Management

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_CMOCKA=y
CONFIG_TESTS_TESTSUITES=y
CONFIG_TESTS_TESTSUITES_STACKSIZE=16384
CONFIG_CM_MM_TEST=y
+CONFIG_ARCH_SETJMP_H=y
+CONFIG_BUILTIN=y
+CONFIG_NSH_BUILTIN_APPS=y
+CONFIG_SCHED_HAVE_PARENT=y
+CONFIG_SCHED_LPWORK=y
```

2. Compile after enabling.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter cmocka_mm_test in nsh>.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.1.2 System Scheduling Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Scheduling

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_CMOCKA=y
CONFIG_TESTS_TESTSUITES=y
CONFIG_TESTS_TESTSUITES_STACKSIZE=16384
CONFIG_CM_SCHED_TEST=y
+CONFIG_ARCH_SETJMP_H=y
+CONFIG_BUILTIN=y
+CONFIG_NSH_BUILTIN_APPS=y
+CONFIG_PSEUDOFS_SOFTLINKS=y
+CONFIG_SCHED_HAVE_PARENT=y
+CONFIG_SCHED_LPWORK=y
```

2. Compile after enabling.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter cmocka_sched_test in nsh>.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.1.3 System Call Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > System Call

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_CMOCKA=y
CONFIG_TESTS_TESTSUITES=y
CONFIG_TESTS_TESTSUITES_STACKSIZE=16384
CONFIG_CM_SYSCALL_TEST=y
+CONFIG_ARCH_SETJMP_H=y
+CONFIG_BUILTIN=y
+CONFIG_NSH_BUILTIN_APPS=y
+ CONFIG_FAT_LFN=y
+CONFIG_IOB_NBUFFERS=128
+CONFIG_IOB_NCHAINS=4
+CONFIG_NET=y
+CONFIG_NETDEV_LATEINIT=y
+CONFIG_NET_ICMP=y
+CONFIG_NET_LOCAL=y
+CONFIG_NET_SOCKOPTS=y
+CONFIG_NET_TCP=y
+CONFIG_NET_UDP=y
+CONFIG_PSEUDOFS_SOFTLINKS=y
+CONFIG_SCHED_HAVE_PARENT=y
+CONFIG_SCHED_LPWORK=y
```

2. Compile after enabling.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter cmocka_syscall_test in nsh>.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.1.4 Kernel-ostest Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_OSTEST=y
```

2. Compile after enabling.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter ostest in nsh.
2. Wait for the execution result.

**Expected Result:**

No error during the test.
ostest_main: Exiting with status 0

---

##### 1.1.5 Kernel-getprime Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_GETPRIME=y
```

2. Compile after enabling.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter getprime in nsh.
2. Wait for the execution result.

**Expected Result:**

getprime took xxx msec

---

##### 1.1.6 Kernel-mm Memory Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_MM=y
```

2. Compile after enabling.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter mm in nsh.
2. Wait for the execution result.

**Expected Result:**

All tests pass and, after the test finishes, the nsh terminal prints TEST COMPLETE.

---

##### 1.1.7 Kernel-scanftest Scan Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_SCANFTEST=y
CONFIG_LIBC_FLOATINGPOINT=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter scanftest in nsh.
2. Wait for the execution result.

**Expected Result:**

After the test finishes, the nsh terminal prints Test /#25 PASSED.

---

##### 1.1.8 Kernel-C Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_EXAMPLES_HELLO=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter hello in nsh.
2. Wait for the execution result.

**Expected Result:**

After the test finishes, the nsh terminal prints Hello, World!!

---

##### 1.1.9 Kernel-Cxx Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_HAVE_CXX=y
CONFIG_EXAMPLES_HELLOXX=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter helloxx in nsh.
2. Wait for the execution result.

**Expected Result:**

After the test finishes, the nsh terminal prints CHelloWorld::HelloWorld

---

##### 1.1.10 Kernel-popen Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_SYSTEM_POPEN=y
CONFIG_EXAMPLES_POPEN=y
CONFIG_DISABLE_POSIX_TIMERS=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter popen in nsh.
2. Wait for the execution result.

**Expected Result:**

After the test finishes, the nsh terminal prints Calling pclose()

---

##### 1.1.11 Kernel-pipe Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_PIPES=y
CONFIG_EXAMPLES_PIPE=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter the following commands in nsh:
pipe
rm /var/testfifo-1
rm /var/testfifo-1
2. Wait for the execution result.

**Expected Result:**

After running the pipe command and the test finishes, the nsh terminal prints Returning success.

---

##### 1.1.12 Kernel-md5 Value Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Add any test file (such as 1.txt) to the etc directory.
2. Enable the following configuration:

```
CONFIG_TESTS_TESTCASES
CONFIG_FS_TEST
CONFIG_FS_TEST_EDONLY
```

3. Compile after enabling the above configuration.
4. Flash the bin package just compiled.
5. Open the nsh window.

**Steps:**

1. Enter md5_test -f /etc/1.txt -c 100 in nsh, where 1.txt is a file in the etc directory.
2. Wait for the execution result.

**Expected Result:**

100 md5 values are obtained, and they are consistent.

---

##### 1.1.13 Kernel-C++ Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.1 System Kernel > Kernel

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_HAVE_CXX=y
CONFIG_EXAMPLES_HELLOXX=y
CONFIG_TESTING_CXXTEST=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter cxxtest in nsh.
2. Wait for the execution result.

**Expected Result:**

After the test finishes, the nsh terminal prints
Test std::vector ============================
v1=1 2 3
s1=Hello, World!
Hello World Good Luck
Test std::map ============================
Test RTTI ============================
extend

---

#### 1.2 System Application

##### 1.2.1 Reboot Startup Exception Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.2 System Application > Boot

**Prerequisites:** 1. Open the nsh window.

**Steps:**

1. Enter reboot in nsh.
2. View the device startup log: check whether there are any errors or other exceptions in the log from the keyword 'reboot' corresponding to the start of boot, to the keyword 'NuttShell (NSH)' corresponding to the completion of system startup.

**Expected Result:**

1. The device reboots successfully.
2. No abnormal errors in the serial port log.

---

##### 1.2.2 Cold boot Startup Exception Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.2 System Application > Boot

**Prerequisites:** 1. Open the nsh window.

**Steps:**

1. Press the reset button on the device.
2. View the device startup log: check whether there are any errors or other exceptions in the log from the start of boot to the keyword 'NuttShell (NSH)' corresponding to the completion of system startup.

**Expected Result:**

1. The device reboots successfully.
2. No abnormal errors in the serial port log.

---

##### 1.2.3 System RAM Resource Usage Statistics

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.2 System Application > Footprint

**Prerequisites:** 1. Open the nsh window.

**Steps:**

1. Enter Free in nsh. If there are multiple cores, enter it on each core.
2. Wait for the execution result.

**Expected Result:**

1. The overall current RAM usage of the system is counted.

---

##### 1.2.4 System Flash Resource Usage Statistics

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.2 System Application > Footprint

**Prerequisites:** 1. Open the nsh window.

**Steps:**

1. Enter df -h in nsh (the vendor needs to provide the hardware Flash resource usage). If there are multiple cores, the usage of each core needs to be provided, and the command needs to be entered on each core.

**Expected Result:**

1. The current hardware Flash resource usage of the system is counted.

---

#### 1.3 Driver BSP

##### 1.3.1 Flashing Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:** 1. Prepare the flashing tool and the version to be flashed.

**Steps:**

1. Refer to the guide document to flash the version to flash.
2. Reboot the device and enter the device nsh terminal.
Note: The flashing guide document and flashing tool are provided by the vendor, and Xiaomi performs acceptance based on the corresponding method.

**Expected Result:**

2. The flashing can be done normally. After flashing succeeds, reboot the device, enter the nsh terminal normally, and the system starts up normally.

---

##### 1.3.2 RAM Read/Write Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_FSTEST
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter fstest -n 10 -m /tmp in nsh.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.3 RAM Read/Write Performance Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_RAMTEST
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.
Note: For the reference document, see

**Steps:**

1. Enter free in nsh to obtain the size of the largest free memory block (largest) currently in the system.
2. Enter ramtest [-w|h|b] -s <size> in nsh and wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.4 RAM Random Read/Write Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_CMOCKA
CONFIG_TESTING_DRIVER_TEST
CONFIG_TESTING_DRIVER_TEST_STACKSIZE=8192
CONFIG_BCH
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.
5. Find the device name corresponding to RAM under /dev on the test platform: ls /dev

**Steps:**

1. Enter the following commands in nsh:
mkrd -m 10 -s 1000 1024
cmocka_driver_block -m /dev/dev_name, where dev_name is filled in based on the actual device name.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.5 Flash Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration: 

```
CONFIG_TESTING_CMOCKA
CONFIG_TESTING_DRIVER_TEST
CONFIG_TESTING_DRIVER_TEST_STACKSIZE=8192
CONFIG_BCH
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.
5. Find the device name corresponding to Flash under /dev on the test platform: ls /dev

**Steps:**

1. Enter cmocka_driver_block -m /dev/dev_name in nsh, where dev_name is filled in based on the actual device name.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.6 GPIO Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration: 

```
CONFIG_TESTING_CMOCKA
CONFIG_TESTING_DRIVER_TEST
CONFIG_TESTING_DRIVER_TEST_STACKSIZE=8192
CONFIG_DEV_GPIO
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.
5. The test platform supports the GPIO driver.
ls /dev -> gpio0
ls /dev -> gpio1
6. Connect the pins corresponding to the two target GPIOs together with a Dupont wire, and complete the test by setting gpio_a and gpio_b. Without configuration, the two addresses default to dev/gpio0 and dev/gpio1.

**Steps:**

1. Enter cmocka_driver_gpio -a /dev/gpio0 -b /dev/gpio1 in nsh.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.7 I2c/Spi Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_CMOCKA=y
CONFIG_TESTING_DRIVER_TEST=y
CONFIG_USENSOR=y
CONFIG_UORB=y
CONFIG_SENSORS=y
CONFIG_SENSORS_BMI160=y
CONFIG_DEBUG_UORB=y
CONFIG_SENSORS_BMI160_I2C=y
CONFIG_SENSORS_BMI160_SPI=y
```

# Enable if testing I2C
# Enable if testing SPI
2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Connect the BMI160 sensor to the board.
5. Open the nsh window.
Note:
1) For CONFIG_SPI or CONFIG_I2C, only one type can be selected for each test; the configurations cannot be enabled together.
2) The BMI160 sensor needs to be enabled at the same time, and the sensor needs to be configured with SPI or I2C to implement the SPI or I2C test. For details, refer to the document.
3) The BMI160 sensor also comes with two sensors, a gyroscope and an accelerometer, which can be used to test that the uORB framework path is normal, replacing the verification of the Sensor part of the driver.

**Steps:**

1. Enter cmocka_driver_i2c_spi in nsh.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.10 Uart Serial Port Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_CMOCKA=y
CONFIG_TESTING_DRIVER_TEST=y
CONFIG_SERIAL=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Use a serial port board to connect the device and the PC:
USB2UART Host
TX -- RX
RX -- TX

**Steps:**

Prefer manual testing; the script has issues that need to be fixed.
| Manual test
1. Enter ls /dev in nsh to query the device name, such as ttyS0.
2. Enter cmocka_driver_uart -d /dev/ttyS0 in nsh.
3. Paste and copy the following content into the serial port and press Enter:
0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ,./<>?;':"[]{}\|!@/#$%^&/*()-+_=
4. Enter 0 and then press Enter.
5. Enter /#
6. Check whether the test PASSes.
| Script test
1. On a computer with the USB-to-UART module plugged in, find the corresponding tty device in the /dev directory: ls /dev/ttyUSB/*, where /* is a number.
2. Execute on the computer terminal:
sudo python3 testing/drivertest/test_content_gen.py /dev/ttyUSB/* (fill in /* based on the actual situation)
3. Enter ls /dev in nsh to query the device name, such as ttyS4.
4. Continue to enter cmocka_driver_uart -d /dev/ttyS4 in nsh and wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.11 Uart File Transfer Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:** 1. Open the nsh window.

**Steps:**

1. Refer to the following document for testing.


**Expected Result:**

Both the file sending and receiving functions work normally.

---

##### 1.3.12 RTC Clock Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_RTC=y
CONFIG_RTC_DRIVER=y
CONFIG_RTC_ARCH=y
CONFIG_RTC_PERIODIC=y
CONFIG_RTC_IOCTL=y
CONFIG_TESTING_DRIVER_TEST=y
CONFIG_TESTING_CMOCKA=y
CONFIG_SIG_EVTHREAD=y
CONFIG_RTC_ALARM=y
CONFIG_DRIVERS_RTC=y
CONFIG_RTC_DATETIME=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Enter cmocka_driver_rtc in nsh.
2. Wait for the execution result.

**Expected Result:**

The test result is PASS, with no exceptions.

---

##### 1.3.13 Timer Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

As the system clock, the Timer is divided into two types: arch alarm and arch timer. You need to determine which type of timer the vendor has adapted. Execute ls /dev to see which device node is registered: arch alarm is /dev/oneshot, and arch timer is /dev/timer.
1. Enable the following configuration:

```
CONFIG_TESTING_DRIVER_TEST=y
CONFIG_TESTING_CMOCKA=y
CONFIG_ONESHOT=y
CONFIG_ALARM_ARCH=y
CONFIG_TIMER=y
CONFIG_TIMER_ARCH=y
```

1) Enable if the vendor adapts arch alarm.
2) Enable if the vendor adapts arch timer.
2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

If the vendor adapts arch alarm:
1. Find the device name corresponding to oneshot under /dev on the test platform: ls /dev
2. Enter cmocka_driver_oneshot -d /dev/oneshot/* in nsh, where /* is a number.
3. Wait for the execution result.
If the vendor adapts arch timer:
4. Find the device name corresponding to oneshot under /dev on the test platform: ls /dev
5. Enter cmocka_driver_oneshot -d /timer/* in nsh, where /* is a number.
6. Wait for the execution result.

**Expected Result:**

3. By default, the test result is output after a 25s delay; the test result is PASS with no exceptions.
6. By default, the test result is output after a 20s delay; the test result is PASS with no exceptions.

---

##### 1.3.14 Time Consistency Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:** 1. The PC enters the device main core through the serial port minicom.
2. The device starts up normally, and ensure that network provisioning has not been performed (the main purpose is to exclude the impact of ntp synchronization on the time).

**Steps:**

1. Enable the minicom time: execute ctrl+a, n, Enter in the serial port.
2. Set the main core consistent with the current PC time, e.g.: date -s "Nov 11 11:11:00 2022"
3. Enter date in nsh.
4. Check the date every 6h (4 times in total).

**Expected Result:**

1. The current PC time is displayed.
2. The setting succeeds, with no errors.
3. The device callback time is consistent with the current PC time.
4. After the device is left standing for more than 24 hours, the difference between the callback time and the current PC time is <=2s.

---

##### 1.3.15 Watchdog Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_WATCHDOG=y
CONFIG_BOARDCTL_RESET_CAUSE=y
CONFIG_TESTING_DRIVER_TEST=y
CONFIG_CMOCKA=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.
Note: When the chip vendor initializes the wdt, it needs to actively call panic in the wdt interrupt, and raise the watchdog interrupt priority to ensure that the watchdog interrupt can interrupt the critical_section, so that the wdt interrupt can be entered even without feeding the dog while interrupts are disabled.

**Steps:**

1. Enter the following commands in nsh in sequence:
cmocka_driver_watchdog -r 0 // Test whether the watchdog takes effect after the timeout is reached
cmocka_driver_watchdog -r 1 // Test interrupting the critical_section, so that the wdt interrupt can be entered even without feeding the dog while interrupts are disabled.
cmocka_driver_watchdog -r 2 // Test whether the watchdog takes effect during an infinite loop after interrupts are enabled
cmocka_driver_watchdog -r 3 // Test normal dog feeding; the result outputs PASS
Note: Execute the command cmocka_driver_watchdog and pass in the parameter through -r. The parameter ranges from 0 to 3, entering 4 different cases respectively. Therefore, the watchdog test needs to be executed four times, with the parameter executed in sequence from 0 to 3 (must be executed in order).

**Expected Result:**

1. Observe that when the -r parameter is 0/1/2, the dog-bite state can actively trigger assert, print the stack information, and reboot, and the reboot reason is BOARDIOC_RESETCAUSE_SYS_RWDT. When the -r parameter is 3, the dog is fed normally and PASS is output.

---

##### 1.3.16 RNG Functional Test


**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_NIST_STS=y
CONFIG_TESTING_DRIVER_TEST=y
CONFIG_CMOCKA=y
```

2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.

**Steps:**

1. Execute as follows:
cd /tmp
mkdir -p experiments/AlgorithmTesting/ApproximateEntropy
mkdir -p experiments/AlgorithmTesting/CumulativeSums
mkdir -p experiments/AlgorithmTesting/Frequency
mkdir -p experiments/AlgorithmTesting/LongestRun
mkdir -p experiments/AlgorithmTesting/OverlappingTemplate
mkdir -p experiments/AlgorithmTesting/RandomExcursionsVariant
mkdir -p experiments/AlgorithmTesting/Runs
mkdir -p experiments/AlgorithmTesting/Universal
mkdir -p experiments/AlgorithmTesting/BlockFrequency
mkdir -p experiments/AlgorithmTesting/FFT
mkdir -p experiments/AlgorithmTesting/LinearComplexity
mkdir -p experiments/AlgorithmTesting/NonOverlappingTemplate
mkdir -p experiments/AlgorithmTesting/RandomExcursions
mkdir -p experiments/AlgorithmTesting/Rank
mkdir -p experiments/AlgorithmTesting/Serial
nist_sts 400000
2. After entering the test program, enter in sequence:
0
/dev/urandom/
1
0
10
1
3. View the result: cat /tmp/experiments/AlgorithmTesting/finalAnalysisReport.txt

**Expected Result:**

P-Value is the cumulative value of the chi-square distribution of all test results. If its value is greater than 0.0001, the random number sample can be considered to have sufficient uniformity and independence. When the value is not random enough, there will be a /* mark after the value and before the test item name to mark the item.

---

##### 1.3.17 Crypto Functional Test

**Test Purpose:** I. General Self-Test Cases > 1. Functional Testing > 1.3 Driver BSP

**Prerequisites:**

1. Enable the following configuration:

```
CONFIG_TESTING_CRYPTO=y
CONFIG_TESTING_CRYPTO_3DES_CBC=y
CONFIG_TESTING_CRYPTO_AES_CBC=y
CONFIG_TESTING_CRYPTO_AES_CTR=y
CONFIG_TESTING_CRYPTO_3DES_XTS=y
CONFIG_TESTING_CRYPTO_HMAC=y
CONFIG_TESTING_CRYPTO_HASH=y
CONFIG_TESTING_CRYPTO_CRC32=y
CONFIG_TESTING_CRYPTO_ECDSA=y
```

1) If the vendor hardware supports the des3cbc algorithm, the configuration needs to be enabled.
2) If the vendor hardware supports the aescbc algorithm, the configuration needs to be enabled.
3) If the vendor hardware supports the aesctr algorithm, the configuration needs to be enabled.
4) If the vendor hardware supports the aesxts algorithm, the configuration needs to be enabled.
5) If the vendor hardware supports the hmac algorithm, the configuration needs to be enabled.
6) If the vendor hardware supports the hash algorithm, the configuration needs to be enabled.
7) If the vendor hardware supports the crc32 algorithm, the configuration needs to be enabled.
8) If the vendor hardware supports the ecdsa algorithm, the configuration needs to be enabled.
2. Compile after enabling the above configuration.
3. Flash the bin package just compiled.
4. Open the nsh window.
Note: The Vela Crypto framework provides these six test applications: des3cbc, aescbc, aesctr, aesxts, hmac, hash, crc32, and ecdsa. The vendor connects the tests based on the algorithms actually implemented by the hardware.

**Steps:**

1. Confirm the algorithm implemented by the vendor hardware, enter the corresponding algorithm name in nsh, and wait for the execution result.
Example: If the vendor hardware implements the des3cbc algorithm, just enter des3cbc in nsh.

**Expected Result:**

The test program runs and finishes normally, and the result ok is displayed.

---

### 2. Performance Testing

#### 2.1 System Application

##### 2.1.3 Cold Boot Startup Time Test

**Test Purpose:** Verify that the system cold boot duration meets the OS standard.

**Prerequisites:** 1. The serial port of the test device works normally.

**Steps:**

1. Power the device off and on for cold boot 10 times, and count the average startup duration from the start of the minicom serial port tool timestamp to the keyword 'NuttShell (NSH)' corresponding to the completion of system startup.
 Note:
1) Enable minicom timestamp printing: ctrl +A +Z +N
2) Arrange the log to print vertically: ctrl +A +U

**Expected Result:**

1. The average duration of 10 cold boots of the device does not exceed 4 seconds.

---

##### 2.1.4 Reboot Startup Time Test

**Test Purpose:** Verify that the system startup duration of a device reboot meets the OS standard.

**Prerequisites:** 1. The serial port of the test device works normally.

**Steps:**

1. Enter reboot in nsh 10 times, and count the average startup duration from the start of reboot at the minicom serial port tool timestamp to the keyword 'NuttShell (NSH)' corresponding to the completion of system startup.
 Note:
1) Enable minicom timestamp printing: ctrl +A +Z +N
2) Arrange the log to print vertically: ctrl +A +Z+U

**Expected Result:**

1. The average duration of 10 reboots of the device does not exceed 6 seconds.

---

### 3. Stability Testing

#### 3.1 System Kernel

##### 3.1.1 12h Standby Stability Test

**Test Purpose:** Verify that the system has no exceptions when the device is in standby for a long time without network provisioning.

**Prerequisites:**

1. Power on the device.
2. Compile a version with the memory monitoring kasan and show_info test tools enabled.
Enable show_info
LOW_RESOURCE_TEST=y
Enable kasan

```
CONFIG_MM_KASAN=y
```

**Steps:**

1. Leave the device standing for 12h and save the serial port log.

**Expected Result:**

1. The system has no errors, crashes, reboots, or other exceptions.

---

## II. Category-Specific Self-Test Cases

### 1. Functional Testing

#### System Application

##### Full Package Upgrade -- Power Off During the avb_verify Process After a Successful Upgrade Boots Into bl2

**Test Purpose:** II. Category-Specific Self-Test Cases > 4. Functional Testing > 4.1 System Application > OTA

**Prerequisites:** 1. The device is powered normally.
2. The device software and hardware version supports downloading the ota package to the device under test via ymodem, Wi-Fi, or the adb push tool.
3. Confirm that the version built into the device is based on the base version of the differential package.
(Note: For example, if the version in the device is old and the differential package to be upgraded is new to old, that is, the version built into the device is not based on the base version of the differential package.)

**Steps:**

1. Use the relevant tool to transfer the differential upgrade package to the /data directory of the device, and change the upgrade package name to 'ota.zip';
Note:
adb : adb push <local ota package> /data
Wi-Fi:
ifup wlan0
wapi mode wlan0 2
wapi psk wlan0 <hotspot password> 3
wapi essid wlan0 <router SSID> 1
renew wlan0
curl -o /data/ota.zip <server link address>/<ota package>
2. Execute reboot recovery. After the 'ota success' keyword appears in the device serial port log, the device reboots. During the avb_verify process when booting into bl2 (power off must occur during the process of covering all avb_verify bin files), power off the device, power it off and on repeatedly 20 times, and check the device serial port log for any abnormal hangs, crashes, etc.
For tool usage, refer to ymodem:


**Expected Result:**

2. Check the device serial port log: during the avb_verify process when booting into bl2, the device is powered off repeatedly, and the system starts up without exceptions after power on.

---

### 4. Compatibility Testing

##### 6.1.1 Router Compatibility

**Test Purpose:** II. Category-Specific Self-Test Cases > 6. Compatibility Testing > 6.1 System Application

**Steps:**

Refer to the router brands and models and the compatibility test case details provided in the attachment [WiFi Compatibility Router List.md](WiFi兼容性路由器列表.md).

**Expected Result:**

The device can successfully establish a STA connection with various models of routers.

---

### 2. Performance Testing

##### 5.1.9 File System Multi-Thread File Read vela_fs_multi_thread_read_test

**Test Purpose:** II. Category-Specific Self-Test Cases > 5. Performance Testing > 5.1 System Application > File System

**Prerequisites:**

1. Enable the following configuration:

