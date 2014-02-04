Memory Error Detectors

This is absolutely not a replacement for the regular usage of a tool such as [Valgrind](http://valgrind.org/), but it does provide somehow similar benefits, with a significant lesser cost. Welcome in your tools repertoire, the widely acclaimed "Address Sanitizer", a Google project hosted at [Code.Google](https://code.google.com/p/address-sanitizer/)

The integration in PaRSEC is quite straightforward, simply adding *"-fsanitize=address -O1 -fno-omit-frame-pointer"* to your different CFLAGS (C and CXX at this point), and using a compatible compiler ([clang](http://clang.llvm.org/) > 3.0) should be enough. Once, everything compiled with the flags run a quick ctest to validate the code.

Any output matching the following pattern "==[0-9]+==ERROR: AddressSanitizer:" is a bad sign and should be acted upon immediately. 