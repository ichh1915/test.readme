# Feedback

## Notes

IMPORTANT: flexop2 duplicates type definitions found in CommonData. There must be something wrong there?

LSL etc are really just synomyms for MOV with a shift op2. (Though possibly not all op2 options are allowed by ARM? Personally I can't see why there should be any limitation)

right rotation for immediates must be by even number of bits (not clear from docs)?

`tests.fsx` stuff should be implemented at top level in common for all instructions except directives.

Syntax Errors do not seem to be handled???

The multiple modules for different instruction types etc is fine though just for convenience i might myself want to put them togetehr a bit to gte longer files to look at

Also, your code lack modularisation: you should create your own file for tests separate from VTest which is my stuff. 


## Success

Very good documentation, with clear delineation of unusual cases. Some lack of clarity in documentation: do your randomised unit tests do multiple random tests? if so what determines the number? I'd be confidant from this testing that your code works if the randomise dtests were in fact repeated enough times. Could nots ee this in documentation nor was it obvious from code.

Minor issue with lack of clarity over where are types defined: or maybe duplicate types. That should be resolved.



## Ambition

This is a decent number of instructions implemented to high quality.


## Testing

I liked a lot about your testing, and you have a good test framework. My problem is that you do not repeat the randomised tests many times to find possible errors? It would be easy to do that, till it is done i'm not certain yoru code works (reply by e-m if in fact you have tested with enough random data to know this).




## Quality

Overall this is good quality.

excellent use of match, but no pipes, no APs (would help parsing) no monadic result processing.

I appreciated the clean code, still it could be significantly improved by writing in a more functional style.

There is quite a bit of common logic between your different instruction classes, I'd like to have seen you make use of that to simplify code.



