Errors & Logging



Don't use log.



## Don't use panic

Don't use panic except in short test programs. It's better to present the user an error and then decide whether you can continue or not. 



## Don't use log.Fatalf or log.Panic

For the same reasons, don't use log.Fatalf or log.Panic except in tests or short programs, because they will halt your program without cleanup. Better to explicitly call os.Exit if that's what you want to do.

