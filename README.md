# Linux VM Notes

## Introduction

This repo is a place for me to write up some notes on the linux 4.6 VM subsystem.

This originally stems from a [set of notes][linux-gorman] I took from the excellent [Understanding the Linux Virtual Memory Manager][amazon-gorman] by [Mel Gorman][gorman] which, while great, targets the 2.4.22 kernel which was released in 2003, somewhat out of date :)

In these notes I am specifically targeting the 4.6 kernel since it is the current mainline version and should remain a sane and stable basis for notes and hacks for the foreseeable future.

Talking of hacks the sister repo to this one, [linux-vm-hacks][vm-hacks], will contain my code and patches intended to help me understand the VM subsystem.

For actually useful tools if ever I produce them, [memutils][memutils] is the place to be.

[linux-gorman]:https://github.com/lorenzo-stoakes/linux-gorman-book-notes
[amazon-gorman]:http://www.amazon.co.uk/Understanding-Virtual-Memory-Manager-Perens/dp/0131453483
[gorman]:http://www.csn.ul.ie/~mel/blog/
[vm-hacks]:https://github.com/lorenzo-stoakes/linux-vm-hacks
[memutils]:https://github.com/lorenzo-stoakes/memutils
