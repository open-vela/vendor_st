NUCLEO-H7A3ZI-Q README
======================

This is the openvela board support for the STMicroelectronics
NUCLEO-H7A3ZI-Q development board (MB1363) based on STM32H7A3ZIT6Q
(Cortex-M7 @ 280 MHz, 2 MB Flash dual-bank, 1.4 MB SRAM).


Configurations
--------------

  nsh       Minimal NSH on USART3 only (~140 KB)
  xts       NSH + xTS Bucket-A1 kernel test suite (~400 KB)
  m2        M2 demo: LED + I2C1/2 + SPI3 + PWM + RTC + Watchdog + ADC1
            (~430 KB)
  m2-bsp    M2 + xTS BSP/driver tests (cmocka_driver_block, _rtc, _pwm,
            _uart, _watchdog) + RNG + progmem MTD (~542 KB)


Build
-----

  cd nuttx
  ./tools/configure.sh -E nucleo-h7a3zi-q:nsh
  make -j$(nproc)


Flash via OpenOCD
-----------------

  openocd -f interface/stlink.cfg -f target/stm32h7x.cfg \
    -c "program nuttx.bin 0x08000000 verify reset exit"


Serial Console
--------------

USART3 PD8/PD9 routed to ST-Link VCP. Default appears as /dev/ttyACM0
or /dev/ttyACM1 depending on enumeration order.

  sudo minicom -D /dev/ttyACM1 -b 115200
  # or
  sudo screen /dev/ttyACM1 115200


Recovery from Bricked State (RDP / DA Lock)
-------------------------------------------

If openocd fails with "init mode failed (unable to connect to the
target)" and STM32CubeProgrammer reports "Unable to get core ID",
the chip is in RDP-locked state. Recovery options (in order):

  Option A: STM32CubeProgrammer GUI -> Option Bytes -> RDP=0xAA -> Apply
            (works for RDP Level 1, performs full chip erase)

  Option B: Pull BOOT0 high (CN11 BOOT0 -> 3V3), re-plug USB,
            then: STM32_Programmer_CLI -c port=USB1 -e all

  Option C: If A and B fail, chip is RDP Level 2 (permanent) - replace board

See openspec/changes/verify-stm32h7a3-round3-fixes/proposal.md for
detailed recovery procedure.


Pin Mapping (UM2408)
--------------------

  USART3 (NSH console):     PD8 (TX), PD9 (RX) - via ST-Link VCP
  USART6 (second uart):     PG14 (TX), PG9 (RX)
  I2C1 (Arduino D14/D15):   PB8 (SCL), PB9 (SDA)
  I2C2 (Arduino D68/D69):   PF1 (SCL), PF0 (SDA)
  SPI3:                     PB3 (SCK), PB4 (MISO), PB5 (MOSI), PA4 (NSS)
  TIM1 PWM ch1:             PE9 (Arduino D6)
  USER LEDs:                LD1=PB0, LD2=PE1, LD3=PB14
  USER button:              PC13 (B1, blue)
  Reset button:             NRST (B2, black)


Clock Plan
----------

  HSE       8 MHz BYPASS (ST-Link MCO)
  PLL1      M=4 N=280 P=2 -> SYSCLK 280 MHz @ VOS0
  HCLK      280 MHz (no division)
  APBx      140 MHz (HCLK/2)
  Flash     LATENCY=6, WRHIGHFREQ=0b11
  RTC       LSE 32.768 kHz
  RNG       HSI48 (M2-BSP only)


Known Limitations
-----------------

  * USB OTG not enabled by default. H7A3 has only OTG_HS controller
    used as FS via embedded HS PHY in FS mode; current openvela
    OTG driver assumes H743 OTG_FS register layout. Captured as
    follow-up: add-stm32h7a3-usb-fs.

  * cxxtest case (xTS 1.1.13) not supported - requires full C++ STL
    (#include <map>); openvela libcxxmini does not provide STL
    containers.

  * cmocka_sched_test, cmocka_syscall_test, md5_test cases (xTS
    1.1.2/1.1.3/1.1.12) not supported in current trunk - source
    files missing from tests/testsuites/kernel/{sched,syscall,md5}.

  * TIM2 oneshot driver may crash on H7A3 specific path - oracle
    investigation found no provable bug; needs real boot crash log
    capture. Captured as follow-up: add-stm32h7a3-oneshot-real-fix.


License
-------

Apache-2.0. See workspace LICENSE file.
