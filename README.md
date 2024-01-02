# worditz
An example Wordle implementation for the console. 

```python3
import os, sys, random, json

CLEAR      = ('clear', 'cls')[sys.platform.startswith('win')]

# game config
MINTURNS = 1
MAXTURNS = 10
MINWORD  = 3
MAXWORD  = 16

# data storage
DAT = os.path.join(os.getcwd(), 'dat')
if not os.path.isdir(DAT):
    os.path.mkdir(DAT)

# stats
playing = True
total   = 0
wins    = 0

while playing:
    os.system(CLEAR)
    print('WORDITZ\n')
    
    #allow user to set the amount of turns they want
    turns = 0
    while not (MINTURNS <= turns <= MAXTURNS):
        while not (u := input(f'Turns {MINTURNS}-{MAXTURNS}: ')).isdigit():
            os.system(CLEAR)
            print('WORDITZ\n')
            
        turns = int(u)
     
    #allow user to select word length
    wordlength = 0
    while not (MINWORD <= wordlength <= MAXWORD):
        while not (u := input(f'Word length {MINWORD}-{MAXWORD}: ')).isdigit():
            os.system(CLEAR)
            print('WORDITZ\n')
            
        wordlength = int(u)
        
    #expected path to words of this length
    wordfile = os.path.join(DAT, f'words_{wordlength}.json')
    
    # get local wordlist
    if os.path.isfile(wordfile):
        with open(wordfile, 'r') as file:
            wordlist = json.load(file)
            
    # create local wordlist
    else:
        import nltk 
        nltk.download('words')
        
        from nltk.corpus import words
        
        wordlist = [word.upper() for word in words.words() if len(word)==wordlength]
        
        with open(wordfile, 'w') as file:
            file.write(json.dumps(wordlist, indent=4))
    
    os.system(CLEAR)
    print(f'WORDITZ: {wordlength} letter word, {turns} chances to solve\n')
    print('round: {}  |  wins: {}  |  losses: {}'.format(total+1, wins, total-wins))
    
    random.shuffle(wordlist)
    
    # prime
    won     = False
    guesses = []
    word    = wordlist.pop(-1)
    
    for i in range(turns):
        user = input('\n').upper()[:wordlength]
        
        if (won := (user == word)): 
            wins += 1
            break
        
        guesses.append([])
        
        for n,c in enumerate(user):
            t  = word.count(c)
            tx = ''.join(guesses[-1]).upper().count(c)
            
            # if the character is in the word and has not already been considerecd
            if (c in word) and (tx < t): 
                # upper if character is in the correct position else lower
                guesses[-1].append((c.lower, c.upper)[word[n] == c]())
            else: 
                # character is not in the word or all instances of the character have already been counted
                guesses[-1].append('-')
             
        print(''.join(guesses[-1]))
        
    print(f'\nword: {word}')
    message = 'You {}! Play again? '.format(('lost', 'won')[won])
    playing = input(message).lower().startswith('y')
    
    total += 1
```
