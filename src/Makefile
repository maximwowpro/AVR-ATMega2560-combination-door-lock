TARGET:=main
DEPS:=eeprom keyboard alarm motor lock uart tools oled led_rgb
MCU:=atmega2560			# see avr-as --help for full list
PROGPORT:=/dev/ttyACM0		# see ls /dev | grep tty and 99-Arduino.rules

CC=avr-gcc
CFLAGS=-mmcu=$(MCU) -Os -std=gnu11 -Wall -Wextra -Wpedantic -Waddr-space-convert -Wmisspelled-isr # -save-temps -Werror  
SIZE:=avr-size --format=avr --mcu=$(MCU)
OBJCOPY:=avr-objcopy -j .text -j .data -O ihex
AVRDUDE:=avrdude

.PHONY: help all clean flash hex

help:				## display this message
	@echo Available options:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

all: clean | flash		## clean, build & flash

clean:				## tidy things up
	-rm -f $(TARGET:=.i) $(TARGET:=.s) $(TARGET:=.o) $(TARGET:=.elf) $(TARGET:=.hex) $(addsuffix .o, $(DEPS)) $(addsuffix .i, $(DEPS)) $(addsuffix .s, $(DEPS))

flash: $(TARGET:=.hex)		## flash MCU with .hex
	$(AVRDUDE) -v -q -V -p$(MCU) -carduino -P$(PROGPORT) -cwiring -D -b115200 -Uflash:w:$<:i

hex: $(TARGET:=.hex)		## create .hex file

gdb: CFLAGS+=-g
gdb: clean | $(TARGET:=.hex)
	
#	simavr -g -m $(MCU) -f 16000000 $(TARGET:=.hex) &
#	avr-gdb --tui -ex="target remote :1234" $(TARGET:=.elf)

$(TARGET:=.elf): $(TARGET:=.c) $(addsuffix .o, $(DEPS))
	-@echo Building \'$(TARGET)\' elf
	$(CC) $(CFLAGS) $(addsuffix .o, $(DEPS)) $(TARGET:=.c) -o $@
	-@echo -en '\033[0;32m'
	$(SIZE) $@
	-@echo -en '\033[0m'

%.hex: %.elf
	$(OBJCOPY) $< $@

%.o : %.c
	$(CC) $(CFLAGS) -c $< -o $@

