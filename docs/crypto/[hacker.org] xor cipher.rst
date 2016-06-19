============================================================================================================
[hacker.org] XOR Cipher
============================================================================================================

XOR Cipher1
============================================================================================================

.. code-block:: python

    _encode = '3d2e212b20226f3c2a2a2b'.decode('hex')

    text = ''
    for l in _encode:
        m = l.encode('hex')
        m = int(m,16)^79
        text += chr(m)

    print text

|

XOR Cipher2
============================================================================================================

.. code-block:: python

    _encode = '948881859781c4979186898d90c4c68c85878f85808b8b808881c6c4828b96c4908c8d97c4878c858888818a8381'.decode('hex')

    for n in range(255):
        text = ''

        for l in _encode:
            m = l.encode('hex')
            m = int(m,16)^n
            text += chr(m)
            
        print text

|

XOR Cipher3
============================================================================================================


.. code-block:: python

    _encode = '31cf55aa0c91fb6fcb33f34793fe00c72ebc4c88fd57dc6ba71e71b759d83588'.decode('hex')

    for x in range(255):
        for y in range(255):
            tmp = x
            text = ''

            for l in _encode:
                m = l.encode('hex')
                m = tmp ^ int(m,16)
                text += chr(m)
                tmp = (tmp+y)%256

            print text



|

Feedback Cipher
============================================================================================================

.. code-block:: python

    _encode = '751a6f1d3d5c3241365321016c05620a7e5e34413246660461412e5a2e412c49254a24'.decode('hex')

    text = ''

    for l in range(len(_encode)-1):
        x = _encode[l].encode('hex')
        y = _encode[l+1].encode('hex')
        m = int(x,16)^int(y,16)
        text += chr(m)

    print text

|

base7 convert
============================================================================================================

.. code-block:: python

    a = 28679718602997181072337614380936720482949

    def convert(n, base):
        T = "0123456789ABCDEF"
        q, r = divmod(n, base)
        if q == 0:
            return T[r]
        else:
            return convert(q, base) + T[r]

    print convert(a, 7)

|

text converter 1
============================================================================================================

.. code-block:: python

    import string

    a = '''
    You must discover what is significant in the following text:
    when i found that i was a prisoner a sort of wild feeling came over me. i rushed up and down the stairs, trying every door and peering out of every window i could find, but after a little the conviction of my helplessness overpowered all other feelings. when i look back after a few hours i think i must have been mad for the time, for i behaved much as a rat does in a trap. when, however, the conviction had come to me that i was helpless i sat down quietly, as quietly as i have ever done anything in my life, and began to think over what was best to be done. i am thinking still, and as Yet have come to no definite conclusion. Of one thing only am i certain. that it is no use making my ideas known to the count. he knows well that i am imprisoned, and as he has done it himself, and has doubtless his own motives for it, he would only deceive me if i trusted him fully with the facts. so far as i can see, my only plan will be to keep my knowledge and my fears to myself, and my eyes open. i am, i know, either being deceived, like a baby, by my own fears, or else i am in desperate straits, and if the latter be so, i need, and shall need, all my brains to get through. i had hardly come to this conclusion when i heard the great door below shut, and knew that the count had returned. he did not come at once into the library, so i went cautiously to my own room and found him making the bed. this was odd, but only confirmed what i had all along thought, that there are no servants in the house. when later i saw him through the chink of the hinges of the door laying the table in the dining room, i was assured of it. for if he does himself all these menial offices, surely it is proof that there is no one else in the castle, it must have been the count himself who was the driver of the coach that brought me here. this is a terrible thought, for if so, what does it mean that he could control the wolves, as he did, by only holding Up his hand for silence? how was it that all the people at bistritz and on the coach had some terrible fear for me? what meant the giving of the crucifix, of the garlic, of the wild Rose, of the mountain Ash? bless that good, good woman who hung the crucifix round my Neck! for it is a comfort and a Strength to me Whenever i touch it. it is odd that a thing which i have been taught to regard with disfavour and as idolatrous should in a time of loneliness and trouble be of help. is it that there is something in the Essence of the thing itself, or that it is a medium, a tangible help, in conveying memories of sympathy and comfort? some time, if it may be, i must examine this matter and try to make up my mind about it. in the meantime i must find out all i can about count dracula, as it may help me to understand. tonight he may talk of himself, if i turn the conversation that way. i must be very careful, however, not to awake his suspicion. midnight.--i have had a long talk with the count. i asked him a few questions on transylvania history, and he warmed up to the subject wonderfully. in his speaking of things and people, and especially of battles, he spoke as if he had been present at them all. this he afterwards explained by saying that to a boyar the pride of his house and name is his own pride, that their glory is his glory, that their fate is his fate. whenever he spoke of his house he always said "we", and spoke almost in the plural, like a king speaking. i wish i could put down all he said exactly as he said it, for to me it was most fascinating. it seemed to have in it a whole history of the country. he grew excited as he spoke, and walked about the Room pulling his great white Moustache and grasping anything on which he laid his hands as though he would crush it by main strength. one thing he said which i shall put down as nearly as i can, for it tells in its way the story of his race. "we szekelys have a right to be proud, for in our veins flows the blood of many brave races who fought as the lion fights, for lordship. here, in the whirlpool of european races, the Ugric tribe bore down from iceland the fighting Spirit which Thor and wodin gave them, which their berserkers displayed to such fell intent on the seaboards of europe, aye, and of asia and africa too, till the peoples thought that the werewolves themselves had Come. here, too, when they came, they found the huns, whose warlike fury had swept the Earth like a living flame, till the dying peoples held that in their veins Ran The blood of those old witches, who, expelled from scythia had mated with the devils in the desert. fools, fools! what devil or what witch was ever so great As attila, whose blood Is in these veins?" he held up his arms. "is it a wonder that we were a conquering race, that we were proud, that when the magyar, the lombard, the avar, the bulgar, or the turk poured his thousands on our frontiers, we drove them back? is it strange that when arpad and his legions swept through the hungarian fatherland he found us here when he reached the frontier, that the honfoglalas was completed there? and when the hungarian flood swept eastward, the szekelys were claimed as kindred by the victorious magyars, and to us for centuries was trusted the guarding of the frontier of turkeyland. aye, and more than that, endless duty of the frontier guard, for as the turks say, 'water sleeps, and the enemy is sleepless.' who more gladly than we throughout the four nations received the 'bloody sword,' or at its warlike call flocked quicker to the standard of the king? when was redeemed that great shame of my Nation, the shame of cassova, when the flags of the wallach and the magyar went down beneath the crescent? who was it but one of my own race who as voivode crossed the danube and beat the turk on his own ground? this was a dracula indeed! woe was it that his own unworthy brother, when he had fallen, sold his people to the turk and brought the shame of slavery on them! was it not this dracula, indeed, who inspired that other of his race who in a Later age again and again brought his forces over the great river into turkeyland, who, when he was beaten back, came again, and again, though he had to come alone from the bloody field where his troops were being slaughtered, since he knew that he alone could ultimately triumph! they said that he thought only of himself. bah! what good are peasants without a leader? where ends the war without a brain and heart to conduct it? again, when, after the battle of mohacs, we threw off the hungarian Yoke, we of the dracula Blood were amongst their leaders, for our spirit would not brook that we were not free. ah, young sir, the szekelys, and the dracula as their heart's blood, their brains, and their swords, can boast a record that mushroom growths like the hapsburgs and the romanoffs can never reach. the warlike days are over. blood is too precious a thing in these days of dishonourable peace, and the glories of the great races are as a tale that is told." it was by this time close on morning, and we went to bed. (mem., this diary seems horribly like the beginning of the "arabian nights," for everything has to break off at cockcrow, or like the ghost of hamlet's father.) 12 may.--let me begin with facts, bare, meager facts, verified by books and figures, and of which there can be no doubt. i must not confuse them with Experiences which will have to rest on my own observation, or my memory of them. last evening when the count came from his room he began by asking me questions on legal matters and on the doing of certain kinds of business. i had Spent the day wearily over books, and, simply to keep my mind occupied, went over some of the matters i had been examined in at lincoln's inn. there was a certain method in the count's inquiries, so i shall try to put them down in sequence. the knowledge may somehow or some time be Useful to me. first, he asked if a man in england might have two solicitors or more. i told him he might have a dozen if he wished, but that it would Not be wise to have more than one Solicitor engaged in one transaction, as only one could act at a time, and that to change would be certain to militate against His Interest. he seemed thoroughly to understand, and went on to ask if there would be any practical difficulty in having one man to attend, say, to banking, and another to look after shipping, in case local help were Needed in a place far from the home of the banking solicitor. i asked to Explain more fully, so that i might not by any chance mislead him, so he said, "i shall illustrate. your friend and mine, mr. peter hawkins, from under the shadow of your beautiful cathedral at exeter, which is far from london, buys for me through your good self my place at london. good! now here let me say frankly, lest you should think it strange that i have sought the services of one so far off from london instead of some one resident there, that my motive was that no local interest might be served save my wish only, and as one of london residence might, perhaps, have some purpose of himself or friend to serve, i went thus afield to seek my agent, whose labours should be only to my interest. now, suppose i, who have much of affairs, wish to ship goods, say, to newcastle, or durham, or harwich, or dover, might it not be that it could with more ease be done by consigning to one in these ports?"
    '''

    answer = ''
    for l in a:
        if l.isupper():
            answer += l

    print answer

|

text converter 2
============================================================================================================

.. code-block:: python

    import string

    a = '''
    in cryptography, a substitution cipher is a method of encryption by which units of plainteXt are suBstituted with ciphertext according to a regular system; the "units" may be single letters (the most common), pairs of letters, triplets of letters, mixtures of the above, and so forth. the receiver deciphers the text by performinG an inverse substitution. substitution ciphers can be compared with tRansposition ciphers. in a transposition cipher, the units of the plaintext are rearranged in a different and usually quite complex order, but the units themselves are left unchanged. by contrast, in a substitution cipher, the units of the plaintext are retained in the same sequence in the ciphertext, but the units themselves are altered. there are a number of different types of substitution cipher. if the cipher operates on single letters, it is termed a simple substitution cipher; a cipher that operates on larger groups of letters is termed polygraphic. a monoalphabetic cipher uses fixed substitution over the entire message, Whereas a polyalphabetic cipher uses a number of substItutions at different times in the message such as with homophones, where a unit from the plaintext is mapped to one of several possibilities in the Ciphertext. substitution over a sinGle letter simple substitution can be Demonstrated by writing out the alphabet in some order to represent the substitution. this is termed a substitution alphabet. the cipher alphabet may be shifted or reversed (creating the caesar and atbash ciphers, respectively) or scrambled in a more complex fashion, in which case it is called a mixed alphabet or deranged alphabet. traditionally, mixed alphabets are created by first writing out a keyword, removing repeated letters in it, then writing all the remaining letters in the alphabet. a disadvantage of this method of derangement is that the last letters of the alphabet (which are mostly low freQuency) tend to stay at the end. a stronger way of constructIng a mixed alphabet is to perform a columnar transposition on the ordinary alphabet using the keyword, but this is not often done. although the number of possible keys is very large (26! = 288.4, or about 88 bits), this Cipher is not very strong, beinG easily broken. provided the message is of reasonable length (see below), the cryptanalyst can deduce the probable meaning of the most common symbols by analyzing the frequency distRibution of the cipherteXt frequency analysis. this allows formation of partial words, which can be tentatively filled in, progressively expanding the (partial) solution (see frequency analysis for a demonstration of this). in some cases, underlying words can also Be determined from the pattern of their letters; for example, attract, osseous, and words with those two as the root are the only common enGlish words with the pattern abbcaDb. many people soLve such ciphers for recReation, as With cryptogram puzzles in the newspaper. accordinG to the unicity distance of english, 27.
    '''

    answer = ''
    for l in a:
        if l.isupper():
            answer += l

    print answer
    _trans = string.maketrans('ABCDEFGHIJKLMNOPQRSTUVWXYZ','AHWREFEHSJKVMNOPIASTUVNTYZ')
    print string.translate(answer,_trans)

|