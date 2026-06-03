# SPI Driver Adaptation and Usage Guide

\[ English | [简体中文](../../../../../zh-cn/device_dev_guide/driver/bus_driver/SPI/SPI.md) \]

## I. Overview

SPI (Serial Peripheral Interface) is a common synchronous serial communication protocol, mainly used for communication between a processor and various peripheral devices. It uses a fixed set of signal lines to transfer data between a master and slave devices:

- **SCLK (Serial Clock)**: Clock signal line, provided by the master, used to synchronize data transfers.
- **MOSI (Master Output, Slave Input)**: Master output, slave input. The master sends data to the slave over this line.
- **MISO (Master Input, Slave Output)**: Master input, slave output. The slave sends data to the master over this line.
- **SS/CS (Slave Select/Chip Select)**: Slave select signal line. The master enables a particular slave by pulling its SS/CS pin low. If there are multiple slaves, each slave needs its own independent SS/CS pin.

SPI has the following transfer characteristics:

- **Master-slave architecture**: SPI uses a master-slave model. There is only one master on the bus, but multiple slaves can be connected.
- **Full-duplex communication**: Data can be transferred bidirectionally between master and slave at the same time.
- **High-speed transfer**: SPI communication rates are usually high, reaching several megabits per second (Mbps), suitable for scenarios with high data-rate requirements.
- **High flexibility**: By configuring the clock polarity and phase (CPOL and CPHA), SPI can work with devices in four different modes. CPOL determines the idle state of the clock line: if 0, the clock line is low when the SPI bus is idle; if 1, the clock line is high when the SPI bus is idle. CPHA determines the sampling moment: if 0, sampling occurs on the first edge of SCLK; if 1, sampling occurs on the second edge of SCLK.

![img](./figures/001.png)

openvela provides two SPI driver frameworks, one for the SPI master and one for the SPI slave. The following sections introduce each framework and its adaptation process.

## II. SPI master

The openvela SPI master framework merely abstracts the control and transfer operations of the SPI master. Understanding the definition of each interface function is enough to know how to operate or adapt an SPI master controller.

### 1. Driver Framework Layers

Before getting hands-on with adaptation, first build an understanding of the overall framework. openvela uses an Upper/Lower Half model to decouple the SPI master. A complete adaptation usually involves three layers:

| Layer | Location | Responsibility | Adapter's focus |
| :--- | :--- | :--- | :--- |
| Driver layer (Lower Half / Southbound) | `arch/<arch>/src/<chip>/<chip>_spi.c` | Implements chip-specific SPI register operations, fills in `struct spi_ops_s`, and exposes the initialization entry `<chip>_spibus_initialize()` | **Core of adaptation**: implement the callbacks in `spi_ops_s` |
| Board layer | `boards/<arch>/<chip>/<board>/src/<board>_bringup.c` | At system startup, calls `<chip>_spibus_initialize()` to obtain the bus handle; implements board-wiring-specific chip select (CS) and status callbacks; calls `spi_register()` as needed to register the character device | Configure CS pins, register the handle as `/dev/spiN` or pass it directly to an upper-layer driver |
| Application layer | Application / kernel upper-layer driver | Accesses the SPI device through the `/dev/spiN` character device (with the SPI tool) or by directly holding the `spi_dev_s` handle | Perform data transfers via the SPI tool or an upper-layer driver |

The data flow between layers is: application layer (or upper-layer driver) → framework access macro (such as `SPI_EXCHANGE`) → the `spi_ops_s` callbacks implemented by the driver layer → operate the SPI controller hardware. The framework itself contains no hardware logic; it only forwards the unified interface macros to the vendor-implemented `ops`.

> Note: Chip select (`select`) and status (`status`) are strongly tied to the GPIO wiring of a specific board. Many chip families (such as STM32, i.MX RT, Kinetis, etc.) place the common SPI logic in the `arch` shared driver and require the **board** to provide `<chip>_spiNselect()` / `<chip>_spiNstatus()` callbacks; some simpler drivers (such as bl602) operate the CS directly inside the driver-layer `select`. This is determined by how the chip driver is implemented; follow the convention of the chosen chip's shared driver during adaptation.

### 2. Access Interfaces

openvela defines the following interfaces for SPI master control and data transfer:

#### 1. SPI_LOCK

```c
#define SPI_LOCK(d,l) (d)->ops->lock(d,l)

/* Parameters
 * d : spi device
 * l : true: Lock spi bus, false: unlock SPI bus
 */
```

This interface locks/unlocks the SPI bus. When multiple SPI slaves are attached to one SPI bus, before accessing one of the slaves you should first call `SPI_LOCK` to gain exclusive access to the bus, and release it after the access is finished.

#### 2. SPI_SELECT

```c
#define SPI_SELECT(d,id,s) ((d)->ops->select(d,id,s))

/* Parameters
 * d  : spi device
 * id : Identifies the device to select
 * s  : true: slave selected, false: slave de-selected
 */
```

This interface selects/deselects an SPI slave. `id` is a `uint32_t` value whose high 16 bits are the SPI device type and whose low 16 bits are the index of that slave within this type of SPI device:

```c
#define SPIDEV_ID(type,index) ((((uint32_t)(type)  & 0xffff) << 16) | \
                                ((uint32_t)(index) & 0xffff))

#define SPIDEVID_TYPE(devid)   (((uint32_t)(devid) >> 16) & 0xffff)
#define SPIDEVID_INDEX(devid)  ((uint32_t)(devid)        & 0xffff)
```

The SPI device types defined in openvela include:

```c
#define SPIDEV_NONE(n)          SPIDEV_ID(SPIDEVTYPE_NONE,          (n))
#define SPIDEV_MMCSD(n)         SPIDEV_ID(SPIDEVTYPE_MMCSD,         (n))
#define SPIDEV_FLASH(n)         SPIDEV_ID(SPIDEVTYPE_FLASH,         (n))
#define SPIDEV_ETHERNET(n)      SPIDEV_ID(SPIDEVTYPE_ETHERNET,      (n))
#define SPIDEV_DISPLAY(n)       SPIDEV_ID(SPIDEVTYPE_DISPLAY,       (n))
#define SPIDEV_CAMERA(n)        SPIDEV_ID(SPIDEVTYPE_CAMERA,        (n))
#define SPIDEV_WIRELESS(n)      SPIDEV_ID(SPIDEVTYPE_WIRELESS,      (n))
#define SPIDEV_TOUCHSCREEN(n)   SPIDEV_ID(SPIDEVTYPE_TOUCHSCREEN,   (n))
#define SPIDEV_EXPANDER(n)      SPIDEV_ID(SPIDEVTYPE_EXPANDER,      (n))
#define SPIDEV_MUX(n)           SPIDEV_ID(SPIDEVTYPE_MUX,           (n))
#define SPIDEV_AUDIO_DATA(n)    SPIDEV_ID(SPIDEVTYPE_AUDIO_DATA,    (n))
#define SPIDEV_AUDIO_CTRL(n)    SPIDEV_ID(SPIDEVTYPE_AUDIO_CTRL,    (n))
#define SPIDEV_EEPROM(n)        SPIDEV_ID(SPIDEVTYPE_EEPROM,        (n))
#define SPIDEV_ACCELEROMETER(n) SPIDEV_ID(SPIDEVTYPE_ACCELEROMETER, (n))
#define SPIDEV_BAROMETER(n)     SPIDEV_ID(SPIDEVTYPE_BAROMETER,     (n))
#define SPIDEV_TEMPERATURE(n)   SPIDEV_ID(SPIDEVTYPE_TEMPERATURE,   (n))
#define SPIDEV_IEEE802154(n)    SPIDEV_ID(SPIDEVTYPE_IEEE802154,    (n))
#define SPIDEV_CONTACTLESS(n)   SPIDEV_ID(SPIDEVTYPE_CONTACTLESS,   (n))
#define SPIDEV_CANBUS(n)        SPIDEV_ID(SPIDEVTYPE_CANBUS,        (n))
#define SPIDEV_USBHOST(n)       SPIDEV_ID(SPIDEVTYPE_USBHOST,       (n))
#define SPIDEV_LPWAN(n)         SPIDEV_ID(SPIDEVTYPE_LPWAN,         (n))
#define SPIDEV_ADC(n)           SPIDEV_ID(SPIDEVTYPE_ADC,           (n))
#define SPIDEV_MOTOR(n)         SPIDEV_ID(SPIDEVTYPE_MOTOR,         (n))
#define SPIDEV_IMU(n)           SPIDEV_ID(SPIDEVTYPE_IMU,           (n))
#define SPIDEV_USER(n)          SPIDEV_ID(SPIDEVTYPE_USER,          (n))
```

#### 3. SPI_SETFREQUENCY

```c
#define SPI_SETFREQUENCY(d,f) ((d)->ops->setfrequency(d,f))

/* Parameters
 * d : spi device
 * f : The SPI frequency requested
 */
```

This interface sets the clock frequency for SPI transfers. It must be called before an SPI transfer begins.

#### 4. SPI_SETDELAY

```c
#ifdef CONFIG_SPI_DELAY_CONTROL
#  define SPI_SETDELAY(d,a,b,c,i) ((d)->ops->setdelay(d,a,b,c,i))
#endif

/* Parameters
 * d : spi device
 * a : The delay between CS active and first CLK
 * b : The delay between last CLK and CS inactive
 * c : The delay between CS inactive and CS active again
 * i : The delay between frames
 */
```

This interface configures the parameters of the SPI clock signal. When the specified SPI controller supports configuring the clock signal produced by the SPI master, this interface can be used to configure it. `CONFIG_SPI_DELAY_CONTROL` must be enabled before use.

#### 5. SPI_SETMODE

```c
#define SPI_SETMODE(d,m) \
  do { if ((d)->ops->setmode) (d)->ops->setmode(d,m); } while (0)

/* Parameters
 * d : spi device
 * m : The SPI mode requested
 */
```

This interface sets the working mode of the SPI master. SPI has four working modes, which openvela defines as follows:

```c
enum spi_mode_e
{
  SPIDEV_MODE0 = 0,     /* CPOL=0 CPHA=0 */
  SPIDEV_MODE1,         /* CPOL=0 CPHA=1 */
  SPIDEV_MODE2,         /* CPOL=1 CPHA=0 */
  SPIDEV_MODE3,         /* CPOL=1 CPHA=1 */
  SPIDEV_MODETI,        /* CPOL=0 CPHA=1 TI Synchronous Serial Frame Format */
};
```

The working mode of the SPI master must match that of the SPI slave.

#### 6. SPI_SETBITS

```c
#define SPI_SETBITS(d,b) \
  do { if ((d)->ops->setbits) (d)->ops->setbits(d,b); } while (0)

/* Parameters
 * d : spi device
 * b : The number of bits in an SPI word.
 */
```

This interface configures the number of bits contained in one transfer word of the SPI master. The SPI master transfers data in units of words, and this interface defines the size of one SPI master transfer.

#### 7. SPI_HWFEATURES

```c
#ifdef CONFIG_SPI_HWFEATURES
#  define SPI_HWFEATURES(d,f) \
  (((d)->ops->hwfeatures) ? (d)->ops->hwfeatures(d,f) : ((f) == 0 ? OK : -ENOSYS))
#else
#  define SPI_HWFEATURES(d,f) (((f) == 0) ? OK : -ENOSYS)
#endif

/* Parameters
 * d : spi device
 * f : H/W feature flags
 */
```

This interface enables specific hardware features of the SPI master. The parameter `f` is a set of hardware feature flags. openvela currently defines the following hardware feature flags:

```c
Bit 0: HWFEAT_CRCGENERATION                    // Hardware CRC generation (must be disabled by default)
Bit 1: HWFEAT_FORCE_CS_INACTIVE_AFTER_TRANSFER // CS rises after every transfer, even if new data is provided immediately
Bit 2: HWFEAT_FORCE_CS_ACTIVE_AFTER_TRANSFER   // CS does not rise automatically after a transfer, even with no data for a long time
Bit 3: HWFEAT_ESCAPE_LASTXFER                  // Currently a hardware capability flag used for SAMV7
Bit 4: HWFEAT_AUTO_CS_CONTROL                  // CS is automatically controlled by the hardware controller with programmable timings
Bit 5: HWFEAT_INVERT_CS_LEVEL                  // Invert the CS level (active high)
Bit 6: HWFEAT_LSBFIRST                         // Transfer LSB first (default is MSB first)
Bit 7: Turn deferred trigger mode on or off.   // Deferred trigger, primarily used for DMA transmission; the DMA only starts once spi_trigger is called

#  ifdef CONFIG_SPI_CRCGENERATION
#    define HWFEAT_CRCGENERATION                     (1 << 0)
#  endif

#  ifdef CONFIG_SPI_CS_CONTROL
#    define HWFEAT_FORCE_CS_CONTROL_MASK             (31 << 1)
#    define HWFEAT_FORCE_CS_INACTIVE_AFTER_TRANSFER  (1 << 1)
#    define HWFEAT_FORCE_CS_ACTIVE_AFTER_TRANSFER    (1 << 2)
#    define HWFEAT_ESCAPE_LASTXFER                   (1 << 3)
#    define HWFEAT_AUTO_CS_CONTROL                   (1 << 4)
#    define HWFEAT_INVERT_CS_LEVEL                   (1 << 5)
#  endif

#  ifdef CONFIG_SPI_BITORDER
#    define HWFEAT_MSBFIRST                          (0 << 6)
#    define HWFEAT_LSBFIRST                          (1 << 6)
#  endif

#  ifdef CONFIG_SPI_TRIGGER
#    define HWFEAT_TRIGGER                           (1 << 7)
#  endif
```

When the SPI master hardware has unique hardware capabilities that need to be enabled, you can use `SPI_HWFEATURES` to configure them, and enable the corresponding control macro at the same time.

#### 8. SPI_STATUS

```c
#define SPI_STATUS(d,id) \
  ((d)->ops->status ? (d)->ops->status(d, id) : SPI_STATUS_PRESENT)

/* Parameters
 * d  : spi device
 * id : Identifies the device to report status on
 */
```

This interface targets the slave device, not the SPI master hardware itself, and is used to obtain the current status of an SPI MMC/SD. The currently supported statuses are:

```c
#define SPI_STATUS_PRESENT     0x01 /* Bit 0=1: MMC/SD card present */
#define SPI_STATUS_WRPROTECTED 0x02 /* Bit 1=1: MMC/SD card write protected */
```

#### 9. SPI_CMDDATA

```c
#ifdef CONFIG_SPI_CMDDATA
#  define SPI_CMDDATA(d,id,cmd) ((d)->ops->cmddata(d,id,cmd))
#endif

/* Parameters
 * d  : spi device
 * id : Identifies the device
 * cmd: TRUE: The following word is a command; FALSE: the following words are data
 */
```

This interface targets the slave device, not the SPI master hardware itself, and tells the slave whether the upcoming data transfer is CMD data or DATA data. It is mainly used in scenarios where the slave has an explicit state switch between CMD and DATA data. `CONFIG_SPI_CMDDATA` must be enabled before use.

#### 10. SPI_SEND

```c
#define SPI_SEND(d,wd) ((d)->ops->send(d,wd))

/* Parameters
 * d  : spi device
 * wd : The word to send.
 */
```

This interface sends one word to the slave. The actual length of the word should match the number of bits set by `SPI_SETBITS`. For example, if `SPI_SETBITS` is set to 8, then `wd` should be 8-bit data; even if 16-bit data is passed in, the SPI master will only send 8 bits to the slave.

#### 11. SPI_EXCHANGE

```c
#ifdef CONFIG_SPI_EXCHANGE
#  define SPI_EXCHANGE(d,t,r,l) ((d)->ops->exchange(d,t,r,l))
#endif

/* Parameters
 * d : spi device
 * t : A pointer to the buffer of data to be sent
 * r : A pointer to the buffer in which to receive data
 * l : The length of data to be exchanged in units of words
 */
```

This interface performs a bidirectional transfer with the slave. During this transfer, the SPI master sends the data in the tx buffer to the peer and simultaneously receives the peer's data into the rx buffer. The data length is in units of the nbits set by `SPI_SETBITS`. When the SPI master and SPI slave support bidirectional transfers, this interface must be implemented and `CONFIG_SPI_EXCHANGE` enabled. A 4-wire SPI usually needs to implement this interface.

#### 12. SPI_SNDBLOCK

```c
#ifdef CONFIG_SPI_EXCHANGE
#  define SPI_SNDBLOCK(d,b,l) ((d)->ops->exchange(d,b,0,l))
#else
#  define SPI_SNDBLOCK(d,b,l) ((d)->ops->sndblock(d,b,l))
#endif

/* Parameters
 * d : spi device
 * b : A pointer to the buffer of data to be sent
 * l : The length of data to send from the buffer in number of words.
 */
```

This interface sends a block of data to the slave, with the length in units of words. When the SPI master and SPI slave support bidirectional transfers (i.e., `CONFIG_SPI_EXCHANGE` is enabled), the `SPI_EXCHANGE` interface is called directly. Otherwise, a unidirectional transfer function must be implemented. A 3-wire SPI can usually only do half-duplex communication and additionally needs to implement `SPI_SNDBLOCK` and `SPI_RECVBLOCK`.

#### 13. SPI_RECVBLOCK

```c
#ifdef CONFIG_SPI_EXCHANGE
#  define SPI_RECVBLOCK(d,b,l) ((d)->ops->exchange(d,0,b,l))
#else
#  define SPI_RECVBLOCK(d,b,l) ((d)->ops->recvblock(d,b,l))
#endif

/* Parameters
 * d : spi device
 * b : A pointer to the buffer in which to receive data
 * l : The length of data that can be received in the buffer in number of words
 */
```

This interface reads a block of data from the slave, with the length in units of words. Same as `SPI_SNDBLOCK`, when the SPI master and SPI slave support bidirectional transfers (i.e., `CONFIG_SPI_EXCHANGE` is enabled), the `SPI_EXCHANGE` interface is called directly.

#### 14. SPI_REGISTERCALLBACK

```c
#define SPI_REGISTERCALLBACK(d,c,a) \
  ((d)->ops->registercallback ? (d)->ops->registercallback(d,c,a) : -ENOSYS)

/* Parameters
 * d : spi device
 * c : The function to call on the media change
 * a : A caller provided value to return with the callback
 */
```

This interface registers a callback with the SPI, mainly provided for media devices to detect media device state transitions. The callback function follows this prototype:

```c
typedef CODE void (*spi_mediachange_t)(FAR void *arg);
```

Of course, if the corresponding device is not a media device, this interface can also be borrowed to implement your own callback to accomplish the desired function.

#### 15. SPI_TRIGGER

```c
#  define SPI_TRIGGER(d) \
  (((d)->ops->trigger) ? ((d)->ops->trigger(d)) : -ENOSYS)

/* Parameters
 * d : spi device
 */
```

This interface actually triggers a previously configured DMA transfer. When the hardware supports the deferred DMA trigger feature, this interface must be implemented and the corresponding macro enabled.

### 3. Driver Adaptation

The openvela SPI framework is only an abstraction of common interfaces. Underneath, it directly calls the vendor-implemented `struct spi_ops_s`. The vendor adapts according to this structure. The meaning of each function is described in the previous section:

```c
//include/nuttx/spi/spi.h

struct spi_ops_s
{
  CODE int      (*lock)(FAR struct spi_dev_s *dev, bool lock);
  CODE void     (*select)(FAR struct spi_dev_s *dev, uint32_t devid,
                  bool selected);
  CODE uint32_t (*setfrequency)(FAR struct spi_dev_s *dev,
                  uint32_t frequency);
#ifdef CONFIG_SPI_DELAY_CONTROL
  CODE int      (*setdelay)(FAR struct spi_dev_s *dev, uint32_t a,
                  uint32_t b, uint32_t c, uint32_t i);
#endif
  CODE void     (*setmode)(FAR struct spi_dev_s *dev, enum spi_mode_e mode);
  CODE void     (*setbits)(FAR struct spi_dev_s *dev, int nbits);
#ifdef CONFIG_SPI_HWFEATURES
  CODE int      (*hwfeatures)(FAR struct spi_dev_s *dev,
                  spi_hwfeatures_t features);
#endif
  CODE uint8_t  (*status)(FAR struct spi_dev_s *dev, uint32_t devid);
#ifdef CONFIG_SPI_CMDDATA
  CODE int      (*cmddata)(FAR struct spi_dev_s *dev, uint32_t devid,
                  bool cmd);
#endif
  CODE uint32_t (*send)(FAR struct spi_dev_s *dev, uint32_t wd);
#ifdef CONFIG_SPI_EXCHANGE
  CODE void     (*exchange)(FAR struct spi_dev_s *dev,
                  FAR const void *txbuffer, FAR void *rxbuffer,
                  size_t nwords);
#else
  CODE void     (*sndblock)(FAR struct spi_dev_s *dev,
                  FAR const void *buffer, size_t nwords);
  CODE void     (*recvblock)(FAR struct spi_dev_s *dev, FAR void *buffer,
                  size_t nwords);
#endif
#ifdef CONFIG_SPI_TRIGGER
  CODE int      (*trigger)(FAR struct spi_dev_s *dev);
#endif
  CODE int      (*registercallback)(FAR struct spi_dev_s *dev,
                  spi_mediachange_t callback, void *arg);
};

struct spi_dev_s
{
  FAR const struct spi_ops_s *ops;
};
```

The following uses a chip named `<chip>` as an example to demonstrate step by step how to adapt an SPI master controller driver from scratch. The example code is modeled on existing drivers in the repository (such as `arch/risc-v/src/bl602/bl602_spi.c`). Replace it with the register operations of your target chip during actual adaptation.

#### Step 1: Implement the callbacks

In the chip driver file `arch/<arch>/src/<chip>/<chip>_spi.c`, implement each callback according to the prototypes in `spi_ops_s`. Among them, `lock`, `select`, `setfrequency`, and `send` are required; the rest are implemented as needed based on hardware capability and enabled configuration. The work each callback must do is as follows:

| Callback | Required | Main work |
| :--- | :--- | :--- |
| `lock` | Yes | Acquire/release the bus mutex to guarantee exclusive access when multiple devices share the bus |
| `select` | Yes | Pull the corresponding slave's CS low/high (some chips implement this at the board layer, see the framework layers note) |
| `setfrequency` | Yes | Configure the SCLK frequency and return the actual effective frequency |
| `setmode` | No | Configure the CPOL/CPHA working mode |
| `setbits` | No | Configure the bit width of one word |
| `send` | Yes | Transfer one word (returns the received word while sending) |
| `exchange` | Conditional | Implemented when `CONFIG_SPI_EXCHANGE` is enabled; performs a bidirectional transfer of a block of data |
| `sndblock`/`recvblock` | Conditional | Implemented when `CONFIG_SPI_EXCHANGE` is not enabled; performs unidirectional send/receive |

```c
/* arch/<arch>/src/<chip>/<chip>_spi.c */

static int      <chip>_spi_lock(FAR struct spi_dev_s *dev, bool lock);
static void     <chip>_spi_select(FAR struct spi_dev_s *dev,
                                  uint32_t devid, bool selected);
static uint32_t <chip>_spi_setfrequency(FAR struct spi_dev_s *dev,
                                        uint32_t frequency);
static void     <chip>_spi_setmode(FAR struct spi_dev_s *dev,
                                   enum spi_mode_e mode);
static void     <chip>_spi_setbits(FAR struct spi_dev_s *dev, int nbits);
static uint32_t <chip>_spi_send(FAR struct spi_dev_s *dev, uint32_t wd);
static void     <chip>_spi_exchange(FAR struct spi_dev_s *dev,
                                    FAR const void *txbuffer,
                                    FAR void *rxbuffer, size_t nwords);

static uint32_t <chip>_spi_setfrequency(FAR struct spi_dev_s *dev,
                                        uint32_t frequency)
{
  FAR struct <chip>_spi_priv_s *priv = (FAR struct <chip>_spi_priv_s *)dev;

  /* Compute and write the divider register based on the input frequency,
   * then return the actual effective frequency.
   */

  ...
  return actual_frequency;
}

/* The remaining callbacks are omitted; all operate this chip's SPI registers */
```

#### Step 2: Fill in the ops table and define the device instance

Attach the implemented callbacks to a `spi_ops_s` table, then use it to initialize `spi_dev_s` (typically embedded as the first member of the chip private structure `<chip>_spi_priv_s`, so that callbacks can reach the private data via a pointer cast):

```c
static const struct spi_ops_s <chip>_spi_ops =
{
  .lock         = <chip>_spi_lock,
  .select       = <chip>_spi_select,
  .setfrequency = <chip>_spi_setfrequency,
  .setmode      = <chip>_spi_setmode,
  .setbits      = <chip>_spi_setbits,
#ifdef CONFIG_SPI_HWFEATURES
  .hwfeatures   = <chip>_spi_hwfeatures,
#endif
  .status       = <chip>_spi_status,
  .send         = <chip>_spi_send,
#ifdef CONFIG_SPI_EXCHANGE
  .exchange     = <chip>_spi_exchange,
#else
  .sndblock     = <chip>_spi_sndblock,
  .recvblock    = <chip>_spi_recvblock,
#endif
  .registercallback = NULL,
};

struct <chip>_spi_priv_s
{
  struct spi_dev_s spi_dev;   /* Must be the first member */
  /* The following is chip private data */
  uint32_t base;              /* Register base address */
  ...
};

static struct <chip>_spi_priv_s <chip>_spi_priv =
{
  .spi_dev =
  {
    .ops = &<chip>_spi_ops
  },
  ...
};
```

#### Step 3: Provide the initialization entry

Expose an initialization function, conventionally named `<chip>_spibus_initialize()`, which takes a bus/port number, performs hardware initialization, and returns the `spi_dev_s` handle for upper layers:

```c
/****************************************************************************
 * Name: <chip>_spibus_initialize
 *
 * Description:
 *   Initialize the specified SPI bus and return the SPI device handle.
 ****************************************************************************/

FAR struct spi_dev_s *<chip>_spibus_initialize(int port)
{
  FAR struct <chip>_spi_priv_s *priv = &<chip>_spi_priv;

  /* Enable clocks, configure pin mux, reset the controller, set the default
   * frequency/mode/bit width, etc.
   */

  ...

  return (FAR struct spi_dev_s *)priv;
}
```

#### Step 4: Register in the board bringup

In the board file `boards/<arch>/<chip>/<board>/src/<board>_bringup.c`, call the initialization entry to obtain the handle and register it as a character device as needed. If the bus is used exclusively by other kernel drivers (such as an onboard FLASH or sensor), you may skip registering the character device and pass the handle directly to that driver's registration function:

```c
#ifdef CONFIG_SPI_DRIVER
  FAR struct spi_dev_s *spi;

  spi = <chip>_spibus_initialize(0);
  if (spi == NULL)
    {
      return -ENODEV;
    }

  ret = spi_register(spi, 0);   /* Register as /dev/spi0 */
  if (ret < 0)
    {
      spierr("ERROR: spi_register failed: %d\n", ret);
    }
#endif
```

At this point an SPI master controller driver is adapted, and you can proceed to the configuration and testing in the next section.

### 4. Usage

#### 1. Enable the configuration

```c
CONFIG_SPI=y
```

Besides enabling the basic configuration, when the hardware supports other hardware features, enable the corresponding macros as needed, such as `CONFIG_SPI_EXCHANGE`.

#### 2. Driver testing

openvela provides an SPI tool at the application layer to test the SPI master driver adaptation. This tool accesses the SPI device as a file node and performs data transfers. openvela provides a driver in the kernel layer that registers a character device in the file system, so that applications can access the SPI device as a file. To use this test program:

1. Configuration

```c
CONFIG_SPI_DRIVER=y
CONFIG_SPI_EXCHANGE=y // SPI_DRIVER depends on CONFIG_SPI_EXCHANGE, i.e. the driver must adapt the SPI_EXCHANGE interface
CONFIG_SYSTEM_SPITOOL=y
```

2. Registration

After the SPI driver is initialized, call the following interface to register it into the file system:

```c
#ifdef CONFIG_SPI_DRIVER
int spi_register(FAR struct spi_dev_s *spi, int bus);
#endif

/* Parameters
 * spi : spi device
 * bus : The SPI bus number.  This will be used as the SPI device minor
 *     number.  The SPI character device will be registered as /dev/spiN
 *     where N is the minor number
 */
```

3. Usage

Once the related configuration is enabled, the SPI tool runs as a command in nsh. Its usage can be queried with `spi help`:

```bash
nsh> spi
nsh> Usage: spi <cmd> [arguments]

Where <cmd> is one of:

  Show help     : ?
  List buses    : bus
  SPI Exchange  : exch [OPTIONS] [<hex senddata>]
  Show help     : help

Where common _sticky_ OPTIONS include:
  [-b bus] is the SPI bus number (decimal).  Default: 0 Current: 2 // bus number
  [-f freq] SPI frequency.  Default: 4000000 Current: 4000000 // frequency
  [-m mode] Mode for transfer.  Default: 0 Current: 0 // SPI mode, 4 modes
  [-u udelay] Delay after transfer in uS.  Default: 0 Current: 0 // delay after each transfer
  [-w width] Width of bus.  Default: 8 Current: 8 // word width, default 8 bits
  [-x count] Words to exchange.  Default: 1 Current: 4  // transfer length
nsh>
```

This tool's test coverage can basically include all the access interfaces, except for `SPI_STATUS`, `SPI_TRIGGER`, and `SPI_REGISTERCALLBACK`, which are associated with the slave device.

## III. SPI slave

Given the master-slave nature of the SPI bus, a device working as an SPI slave can only passively receive and send data. openvela divides the SPI slave driver framework into two layers: the controller layer and the device layer:

![img](./figures/002.png)

The device layer calls the controller-layer interfaces to send and query data, and the controller layer reads the data it received into the device layer through the device-layer interfaces. A driver based on an SPI slave controller needs to adapt the controller layer, and a device driver built on top of it needs to adapt the device layer.

### 1. Driver Framework Layers

The SPI slave layering differs slightly from the master; it consists of two mutually bound interfaces. Understanding "who calls whom" is the key to adapting an SPI slave:

| Layer | Implemented by | Provided interface | Role |
| :--- | :--- | :--- | :--- |
| Controller layer | The lower-half driver of the SPI slave controller, located at `arch/<arch>/src/<chip>/<chip>_spi_slave.c` | `struct spi_slave_ctrlrops_s` (`bind`/`unbind`/`enqueue`/`qfull`/`qflush`/`qpoll`) | Directly operates the SPI slave controller hardware, responsible for the actual sending/receiving of data |
| Device layer | The device driver built on top of the slave (such as the character device `spi_slave_driver.c`, or a custom protocol device) | `struct spi_slave_devops_s` (`select`/`cmddata`/`getdata`/`receive`/`notify`/`getrecvbuf`) | Invoked as callbacks by the controller layer to process received data and provide data to be sent |

The call direction between the two layers is **bidirectional**:

- **Device layer → controller layer**: The device driver hands data to be sent to the controller via `SPIS_CTRLR_ENQUEUE`, and drives the controller to flush received data back to itself via `SPIS_CTRLR_QPOLL`.
- **Controller layer → device layer**: When the controller detects chip select, receives data, or completes a transfer, it in turn calls back the device layer's `select`/`receive`/`notify` methods.

The binding is established via `SPIS_CTRLR_BIND`: when the device driver initializes, it passes its own `spi_slave_dev_s` to the controller. Once bound, the controller is "armed" and ready to respond at any time to a transfer initiated by the peer master.

> Tip: The controller layer and the device layer usually need to be adapted **together**. If you only want to verify whether the controller-layer driver works, you can directly reuse openvela's built-in character-device device-layer driver (`CONFIG_SPI_SLAVE_DRIVER`, see the "Usage" section of this chapter), without implementing the device layer yourself.

### 2. Controller-Layer Interfaces

#### 1. SPIS_CTRLR_BIND

```c
#define SPIS_CTRLR_BIND(c,d,m,n) ((c)->ops->bind(c,d,m,n))

/* Parameters
 * c : SPI Slave controller interface instance
 * d : SPI Slave device interface instance
 * m : The SPI Slave mode requested
 * n : The number of bits requested.
 *     If value is greater than 0, then it implies MSB first
 *     If value is less than 0, then it implies LSB first with -nbits
 */
```

This interface binds the device and the controller. Inside it, the SPI slave's mode, nbits, and MSB/LSB hardware configuration must be completed, and the SPI slave controller enabled at the same time. The mode and nbits passed when calling this interface must match the configuration of the peer SPI master.

#### 2. SPIS_CTRLR_UNBIND

```c
#define SPIS_CTRLR_UNBIND(c) ((c)->ops->unbind(c))

/* Parameters
 * c : SPI Slave controller interface instance
 */
```

This interface unbinds the controller. When unbinding, the SPI controller should be disabled.

#### 3. SPIS_CTRLR_ENQUEUE

```c
#define SPIS_CTRLR_ENQUEUE(c,v,l)  ((c)->ops->enqueue(c,v,l))

/* Parameters
 * c : SPI Slave controller interface instance
 * v : Pointer to the command/data mode data to be shifted out.
 * l : Number of units of "nbits" wide to enqueue
 */
```

This interface sends data into the controller. The data will be sent out by the SPI slave controller the next time the SPI master initiates a transfer.

#### 4. SPIS_CTRLR_QFULL

```c
#define SPIS_CTRLR_QFULL(c)  ((c)->ops->qfull(c))

/* Parameters
 * c : SPI Slave controller interface instance
 */
```

This interface queries whether the controller's transmit buffer is full.

#### 5. SPIS_CTRLR_QFLUSH

```c
#define SPIS_CTRLR_QFLUSH(c)  ((c)->ops->qflush(c))

/* Parameters
 * c : SPI Slave controller interface instance
 */
```

This interface clears the contents of the controller's transmit buffer.

#### 6. SPIS_CTRLR_QPOLL

```c
#define SPIS_CTRLR_QPOLL(c)  ((c)->ops->qpoll(c))

/* Parameters
 * c : SPI Slave controller interface instance
 */
```

This interface queries the length of data in the controller's receive buffer. If the controller buffer contains data, the controller layer needs to call the device-layer interface `SPIS_DEV_RECEIVE` to read the received-buffer data into the device layer. openvela builds the process of the device reading the controller-layer receive-buffer data into `SPIS_CTRLR_QPOLL`, rather than first calling `SPIS_CTRLR_QPOLL` and then calling `SPIS_DEV_RECEIVE` to read.

### 3. Device-Layer Interfaces

#### 1. SPIS_DEV_RECEIVE

```c
#define SPIS_DEV_RECEIVE(d,v,n)  ((d)->ops->receive(d,v,n))

/* Parameters
 * d : SPI Slave device interface instance
 * v : Pointer to the new data that has been shifted in
 * n : Length of the new data in units of nbits wide
 */
```

This interface receives the data in the controller-layer receive buffer into the device layer. The parameter `v` points to the controller-layer receive buffer address, the parameter `n` is the length of valid data contained in the buffer, and the return value is the actual length of data received into the device layer, in units of the nbits set by `SPIS_CTRLR_BIND`. When the device layer calls `SPIS_CTRLR_QPOLL`, the controller layer should call `SPIS_DEV_RECEIVE` multiple times inside that function to return the received data to the device layer, until the controller-layer receive buffer is empty, or until the return value of `SPIS_DEV_RECEIVE` is less than the passed-in parameter `n` (meaning the controller layer can no longer accept more data); only then does `SPIS_CTRLR_QPOLL` return.

#### 2. SPIS_DEV_GETDATA

```c
#define SPIS_DEV_GETDATA(d,v)  ((d)->ops->getdata(d,v))

/* Parameters
 * d : SPI Slave device interface instance
 * v : Pointer to the data buffer pointer to be shifted out
 */
```

This interface obtains the data to be sent from the device layer. This data will be sent out the next time the SPI master clock arrives.

#### 3. SPIS_DEV_GETRECVBUF

```c
#define SPIS_DEV_GETRECVBUF(d,b)  ((d)->ops->getrecvbuf(d,b))

/* Parameters
 * d : SPI Slave device interface instance
 * b : Pointer to the receive buffer pointer to be shifted in
 */
```

This interface supports zero-copy (nocopy) transfers. The device layer calls it when it wants to perform a zero-copy transfer; if the buffer pointer the controller obtains is non-NULL, the enqueued data is transferred directly into that buffer. The return value is the number of data units that can be received this time.

#### 4. SPIS_DEV_NOTIFY

```c
#define SPIS_DEV_NOTIFY(d,s)  ((d)->ops->notify(d,s))

/* Parameters
 * d : SPI Slave device interface instance
 * s : The Receive and send state, type of state is spi_slave_state_t
 */
```

This interface notifies the device layer that a send or receive process has completed. The device layer can continue to send or receive data according to the state returned by the controller:

```c
typedef enum
{
  SPISLAVE_RX_COMPLETE = 0,
  SPISLAVE_TX_COMPLETE,
  SPISLAVE_TRANSFER_FAILED
} spi_slave_state_t;
```

#### 5. SPIS_DEV_SELECT

```c
#define SPIS_DEV_SELECT(d,s) ((d)->ops->select(d,s))

/* Parameters
 * d : SPI Slave device interface instance
 * s : Indicates whether the chip select is in active state
 */
```

This interface notifies the device layer of a chip-select event detected by the controller.

#### 6. SPIS_DEV_CMDDATA

```c
#define SPIS_DEV_CMDDATA(d,i) ((d)->ops->cmddata(d,i))

/* Parameters
 * d : SPI Slave device interface instance
 * i : True: Data is selected, False: Cmd is selected
 */
```

This interface notifies the device layer of a CMD/DATA state switch.

### 4. Driver Adaptation

SPI slave driver adaptation generally refers to the controller layer, which is responsible for SPI data transfer. The device driver built on top of the SPI slave needs to adapt the device layer. The two are often tightly coupled and need to be adapted together; implement them according to the interfaces provided by the SPI slave. The vtables each layer must implement are as follows:

```c
//include/nuttx/spi/slave.h

struct spi_slave_ctrlrops_s
{
  CODE void     (*bind)(FAR struct spi_slave_ctrlr_s *ctrlr,
                        FAR struct spi_slave_dev_s *sdev,
                        enum spi_slave_mode_e mode, int nbits);
  CODE void     (*unbind)(FAR struct spi_slave_ctrlr_s *ctrlr);
  CODE int      (*enqueue)(FAR struct spi_slave_ctrlr_s *ctrlr,
                           FAR const void *data, size_t nwords);
  CODE bool     (*qfull)(FAR struct spi_slave_ctrlr_s *ctrlr);
  CODE void     (*qflush)(FAR struct spi_slave_ctrlr_s *ctrlr);
  CODE size_t   (*qpoll)(FAR struct spi_slave_ctrlr_s *ctrlr);
};

struct spi_slave_devops_s
{
  CODE void     (*select)(FAR struct spi_slave_dev_s *sdev, bool selected);
  CODE void     (*cmddata)(FAR struct spi_slave_dev_s *sdev, bool data);
  CODE size_t   (*getdata)(FAR struct spi_slave_dev_s *sdev,
                           FAR const void **data);
  CODE size_t   (*receive)(FAR struct spi_slave_dev_s *sdev,
                           FAR const void *data, size_t nwords);
  CODE void     (*notify)(FAR struct spi_slave_dev_s *sdev,
                          spi_slave_state_t state);
  CODE size_t   (*getrecvbuf)(FAR struct spi_slave_dev_s *sdev,
                              FAR void **buffer);
};
```

The following uses a chip named `<chip>` as an example to demonstrate controller-layer adaptation step by step. The example is modeled on existing slave controller drivers in the repository (such as `arch/risc-v/src/common/espressif/esp_spi_slave.c` and `arch/arm/src/samv7/sam_spi_slave.c`). Replace it with the register operations of your target chip during actual adaptation.

#### Step 1: Implement the controller-layer callbacks

In `arch/<arch>/src/<chip>/<chip>_spi_slave.c`, implement each callback according to the prototypes in `spi_slave_ctrlrops_s`. The work each callback must do is as follows:

| Callback | Main work |
| :--- | :--- |
| `bind` | Save the device-layer handle `sdev`, complete the mode, nbits, and MSB/LSB hardware configuration per the arguments and enable the controller; usually call the device-layer `getdata` once here to prefetch the first word to send |
| `unbind` | Release the binding with the device layer, disable the controller, and restore the initial state |
| `enqueue` | Write the data to send provided by the device layer into the transmit queue/buffer, returning the number of data units successfully enqueued |
| `qfull` | Return whether the transmit queue is full |
| `qflush` | Clear the data in the transmit queue that has not yet been sent |
| `qpoll` | Hand the data in the receive buffer back to the device layer by calling its `receive`, returning the number of data units remaining in the queue |

```c
/* arch/<arch>/src/<chip>/<chip>_spi_slave.c */

static void   <chip>_spislave_bind(FAR struct spi_slave_ctrlr_s *ctrlr,
                                   FAR struct spi_slave_dev_s *sdev,
                                   enum spi_slave_mode_e mode, int nbits);
static void   <chip>_spislave_unbind(FAR struct spi_slave_ctrlr_s *ctrlr);
static int    <chip>_spislave_enqueue(FAR struct spi_slave_ctrlr_s *ctrlr,
                                      FAR const void *data, size_t nwords);
static bool   <chip>_spislave_qfull(FAR struct spi_slave_ctrlr_s *ctrlr);
static void   <chip>_spislave_qflush(FAR struct spi_slave_ctrlr_s *ctrlr);
static size_t <chip>_spislave_qpoll(FAR struct spi_slave_ctrlr_s *ctrlr);

static void <chip>_spislave_bind(FAR struct spi_slave_ctrlr_s *ctrlr,
                                 FAR struct spi_slave_dev_s *sdev,
                                 enum spi_slave_mode_e mode, int nbits)
{
  FAR struct <chip>_spislave_priv_s *priv =
    (FAR struct <chip>_spislave_priv_s *)ctrlr;
  FAR const void *data = NULL;

  priv->sdev = sdev;            /* Save the device-layer handle for later callbacks */

  /* Configure the hardware mode/bit width, enable controller interrupts, etc. */

  ...

  /* Prefetch the first word to send, "prime the pump" */

  SPIS_DEV_SELECT(sdev, true);
  SPIS_DEV_GETDATA(sdev, &data);
  ...
}

/* The remaining callbacks are omitted; all operate this chip's SPI slave registers */
```

When the controller receives data in an interrupt, detects a chip-select change, or completes a transfer, it needs to call back the device-layer interfaces. The commonly used ones are: `SPIS_DEV_SELECT` (chip-select change), `SPIS_DEV_RECEIVE` (hand back received data), and `SPIS_DEV_NOTIFY` (notify transfer completion).

#### Step 2: Fill in the ops table and define the controller instance

Attach the callbacks to a `spi_slave_ctrlrops_s` table and embed it in the controller private structure (`spi_slave_ctrlr_s` must be the first member):

```c
static const struct spi_slave_ctrlrops_s <chip>_spislave_ops =
{
  .bind    = <chip>_spislave_bind,
  .unbind  = <chip>_spislave_unbind,
  .enqueue = <chip>_spislave_enqueue,
  .qfull   = <chip>_spislave_qfull,
  .qflush  = <chip>_spislave_qflush,
  .qpoll   = <chip>_spislave_qpoll,
};

struct <chip>_spislave_priv_s
{
  struct spi_slave_ctrlr_s ctrlr;          /* Must be the first member */
  FAR struct spi_slave_dev_s *sdev;        /* Device-layer handle saved at bind */
  /* The following is chip private data: register base, tx/rx buffers, etc. */
  ...
};

static struct <chip>_spislave_priv_s <chip>_spislave_priv =
{
  .ctrlr = { .ops = &<chip>_spislave_ops },
  ...
};
```

#### Step 3: Provide the controller initialization entry

Expose an initialization function, conventionally named `<chip>_spislave_ctrlr_initialize()`, which takes a port number and returns a `spi_slave_ctrlr_s` handle:

```c
/****************************************************************************
 * Name: <chip>_spislave_ctrlr_initialize
 *
 * Description:
 *   Initialize the specified SPI slave controller and return the controller
 *   handle.
 ****************************************************************************/

FAR struct spi_slave_ctrlr_s *<chip>_spislave_ctrlr_initialize(int port)
{
  FAR struct <chip>_spislave_priv_s *priv = &<chip>_spislave_priv;

  /* Enable clocks, configure pin mux, reset the controller, register
   * interrupts, etc.
   */

  ...

  return (FAR struct spi_slave_ctrlr_s *)priv;
}
```

#### Step 4: Register in the board bringup

In the board file, call the initialization entry to obtain the controller handle, then call `spi_slave_register()` to bind the built-in character-device device layer and register it as `/dev/spislvN`:

```c
#ifdef CONFIG_SPI_SLAVE_DRIVER
  FAR struct spi_slave_ctrlr_s *ctrlr;

  ctrlr = <chip>_spislave_ctrlr_initialize(0);
  if (ctrlr == NULL)
    {
      return -ENODEV;
    }

  ret = spi_slave_register(ctrlr, 0);   /* Register as /dev/spislv0 */
  if (ret < 0)
    {
      spierr("ERROR: spi_slave_register failed: %d\n", ret);
    }
#endif
```

To interface with a custom device layer (rather than the built-in character device), have your device driver construct a `spi_slave_dev_s` and call `SPIS_CTRLR_BIND` to bind with the controller.

### 5. Usage

#### 1. Enable the configuration

```c
CONFIG_SPI_SLAVE=y
```

#### 2. Driver testing

openvela provides a device-layer driver that registers a character device in the file system, so that applications can access the SPI slave device as a file, allowing simple testing of the controller-layer driver. To use this test program:

1. Configuration

```c
CONFIG_SPI_SLAVE_DRIVER=y
CONFIG_SPI_SLAVE_DRIVER_MODE=0  // The working mode of the SPI slave, can be configured as 0/1/2/3 as needed
CONFIG_SPI_SLAVE_DRIVER_WIDTH=8 // The nbits of the SPI slave, also configured as needed
CONFIG_SPI_SLAVE_DRIVER_BUFFER_SIZE // The receive buffer size of the SPI character device, allocated as needed
```

2. Registration

After the SPI slave driver is initialized, call the following interface to register it into the file system. The registered character device node is `/dev/spislvN` (N is the minor number):

```c
#ifdef CONFIG_SPI_SLAVE_DRIVER
int spi_slave_register(FAR struct spi_slave_ctrlr_s *ctrlr, int bus);
#endif
```
