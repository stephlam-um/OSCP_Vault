# tee  
command | tee file.txt # save + display output  
command | tee -a file.txt # append instead of overwrite  
command | tee file.txt | grep ssh
# grep
grep "word" file.txt
grep -i "word" file.txt         # case insensitive
grep -r "word" .                # recursive
grep -n "word" file.txt         # line numbers
grep -E "a|b|c" file.txt        # multiple patterns
command | grep "word"           # filter output

# rg (ripgrep)
rg "word"
rg -i "word"
rg -uu "word"                   # include hidden/ignored files
rg -l "word"                    # filenames only
rg -w "word"                    # exact word
rg "user.*pass"                 # regex
rg "word" -t py                 # only python files

# less
less file.txt

# inside less
/word                           # search forward
?word                           # search backward
n                               # next result
N                               # previous result
g                               # top
G                               # bottom
q                               # quit

# useful
rg -i "password|token|apikey|secret"
cat file.txt | grep open