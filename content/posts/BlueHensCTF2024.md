---
title: BlueHensCTF 2024 Writeup
date: 2024-11-11T17:05:11+07:00
draft: false
description:
isStarred: false
---


## Pythagorean Triple

### Brief code explanation
- vector is a list with 3 elements
- matrix is a list with 3 vectors
- dotp() is the dot product of 2 vectors 
- mxv is matrix $\times$ vector, performs dotp for every vector in the matrix to the vector
- The loop perform mxv of a random matrix in the list all with tmp 3000 times and the name of the chosen matrix is written to key
- The key is being used to encrypt the flag using AES, which can be easily decrypted if we know the key (AES is a symmetric encryption)
- We get the tmp and the ciphertext

### TL;DR 
Find the key from tmp then decrypt the ciphertext

### Solution 
To find the key, we have to find a way to determine which matrix in All a vector is a result of mxv with another vector.

Let's assume :
- a vector v2 is the result of mxv(all\[ktmp] , v1). where ktmp is either 'A', 'B', or 'C'.
- v1 = \[x1, y1, z1] and v2 = \[x2, y2, z2]

Then, let's divide ktmp to 3 cases : 
- ktmp = 'A'
According to the code : 
- x2 = x1 - 2y1 + 2z1
- y2 = 2x1 - y1 + 2z1
- z2 = 2x1 - 2y1 + 3z1
But what is x1, y1, and z1 ? 
With some algebra manipulation, we will get 
- x2 + 2y2 - 2z2 = (x1 - 2y1 + 2z1) + 2(2x1 - y1 + 2z1) - 2(2x1 - 2y1 + 3z1) = x1
- 2x2 + y2 - 2z2 = 2(x1 - 2y1 + 2z1) + (2x1 - y1 + 2z1) - 2(2x1 - 2y1 + 3z1) = -y1
- 2x2 + 2y2 - 3z2 = 2(x1 - 2y1 + 2z1) + 2(2x1 - y1 + 2z1) - 3(2x1 - 2y1 + 3z1) = z1
To make it tidier, the 3 equations above is equal to : 
- x1 = x2 + 2y2 - 2z2
- -y1 =  2x2 + y2 - 2z2
- -z1 =  2x2 + 2y2 - 3z2
- ktmp = 'B'
Using the same ways as before we will get : 
- x1 = x2 + 2y2 - 2z2
- y1 =  2x2 + y2 - 2z2
- -z1 =  2x2 + 2y2 - 3z2
- ktmp = 'C'
Again : 
- -x1 = x2 + 2y2 - 2z2
- y1 =  2x2 + y2 - 2z2
- -z1 =  2x2 + 2y2 - 3z2

There is a pattern. 
If the ktmp = 'A', y1 will be negative. 
else If the ktmp = 'C', x1 will be negative. 
else If the ktmp = 'B', none of them will be negative. 

So this is the solver script 

```
import hashlib
from Crypto.Cipher import AES

def dotp(v1,v2):
    return sum([i*j for (i, j) in zip(v1, v2)])
def mxv(mat,vec):
    return [dotp(mat[0], vec), dotp(mat[1], vec), dotp(mat[2],vec)]

ansa = [
    [1, 2, -2],
    [2, 1, -2],
    [2, 2, -3]
]

#the tmp we got from the web
lasttmp = [23746489403642286759537040347676492201199266046801298403395135489170610756871479095507090635058007439275482861945278189796356710758374352145450707511020751123618648698883277084465688937261728619171035162908825416170770470296864629869017615892334848674786817186039211037963691116959652817768850941554691716145583642527105425245416849743654175586241279436486349131391592932848301350283703826293114876027704465049786393881152089437068809904031546708503537227231038617740470084156956130475760124672437005183769753600845283167167941756482826672991114255207977378003577251250523666033785105461014284120858839105926450992249067404573149651842273251624120163437178669926935613909641654911560743756310550480094761877025280576177942827634449148162225848276519240549571407971191859668200659668672546393912300794418171050231646656731085093939039534588012160342830887102718388522467845468266374870489530590720812332932301654059397383471479450535932247634952755492991444516321795793510148332670955009615617331763598597638151836980112059388488144881579492125318261679720303223123358807097253498809264365282565702435075010878960751254584251716250186854088966124540459482981670101400122480991470198052872075428865705186470177429907008526493878504485532332178326598464206157535248307945719585365243532999260902377026419790557956006399535435526200550589425234608323192176249220997315893479378285192244060137370347069396595445007858565294823269345361007270420558749410338839891844242404967653862569088893289585267657581167995667861303748817324116617296905978264528759157064057938444633641079361842452820134042587911959119913578366256583827719872986508187552613897794578886352924094542113829745190835, 21079650979858182701625603873288744241692398914665668510623693959529270669836690486357622734567486628866993669672728694469698966968193094646885798907337406363343225749073918276446899711806380075890546024299888270565799146856781868177984349369929484997913990919739442960923731292498078964399240061098810111039465339141841691771488347512103050084021258308352997360201064647716678894458034709686594638460519898489513381899797694213674801199372395678728533234422438721646977703643884463240431693275857557991381891552673352519992153524011673183097018410843516694797758039028628734268090965213034923663567031515294095064234142894461410685145367922012823164164102942026577704705302695519987276770075049716436732478249816366405939034298599204984632821691752078589937947419122155244249880454709107989611727669610793779749434588195926302810097401740058193133784988466575630741685132827237629420031576185194866727142520854434594377200733206105573930552595943585876693800290437745645045014735074547956119426746030035503233868064016020605066493960896710733513741699883331530795835484719032501897495017060646658628654613280217898976957254892275735282740780197822415886668394146069455168593263312609057867325090339963463362283528072862376041728353898316279252260582442600145348944277208383309804288181350466931480901349901924134599102379362289601389590853260354703939126125150462279359829196089474001593139538738677327163163009329515572971366737917661452394229754197546779829986386155388119376811677545768343797140370800794668989632558072298440405417776921080324148825933205501025180160177210144915650054638389928117389136070149535579185856357662911595497520631842119717437751633884978922477628, 31752912377133714661073343486310800703898130745412318234295748797030044215063029579249367349319668494753693634464654444579013069048974563079009570988534594606741073060314218275098018070164676001844973071851100890706774913154555762928599799741020629653067804882507997153283628578631926452489353890351233307182842096881311900015277092933150535580830027379854398723662274460309905637210208907845848818651387401327755003793924771419256666518284204535282161030020866888302085891111286758536103223590171179027116810338795161432801259037265191579457623997894363808896649407524417977257429584265074809544603351762671117965754379964184034551154317328622801994719016300655340050895443643459108803412285575485886863690485858259462734237855073356594631698877157020751468036108346892620013389767874552820713663398209884585786752407289441734603625713764207144605908511536771916334929569218455614048834444767522269212897160878210562140365458502855266923940161006598986862179999382958881654698392556071998622533114150361456140286761933953051943705972695567203613350571292561008064289422549735149918809924916946219084000828790505791900478552093884126172683873693463233526502440517269564556306446328260291786586150274369669008031459096466608510282425184655060477962939912018500310046021354473782247584391000927603683717154177445430024216866479099733049260954710226811211322711919960136782856559309543523758442067269039064341854695316672072107952704456303986303759320161266235782013826241158508888909683398018450820571183500560904379533925066827623498193626655749055874215966863841689417864417942054868878221611250789394862546673844158442983447663750325813466291062796922436327889236957121458096853]

key=""
for _ in range(3000):
    tmp = mxv(ansa, lasttmp)
    if tmp[0] < 0:
        key = 'C' + key
    elif tmp[1] < 0:
        key = 'A' + key
    else:
        key = 'B' + key
    lasttmp = [abs(i) for i in tmp]

ct = bytes.fromhex('12fe177086c2a9a2716f231ec183a37e036229e1ef9f2df1d0de05f5f3332f8d296002b994099bca')
flag = b"REDACTEDFLAG"
IV = b"0123456789abcdef"
cipher = AES.new(hashlib.sha256(key.encode()).digest(), AES.MODE_OFB, IV)
ct = cipher.decrypt(ct)
print(ct)
```

## CTR Mode is Just XOR

### Brief Code Explanation 
- The code get a plaintext from us
- Then it will return the ciphertext of the given plaintext using AES mode ECB, the nonce (probiv, or initialization vector) for the AES CTR, and the AES CTR encrypted flag
- Note that the plaintext is padded before encrypted and by default it will use PKCS5(or PKCS7), which will always pad no matter what. If the plaintext is 16 bytes, the padding will make it 32
- nonce = '475045713653717a7936644c6d654d'

### Solution
A block in block cipher sized 16 bytes and the encrypted flag is 64 bytes, so the AES mode CTR encrypted 4 blocks of plaintext

From this wikipedia https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation, CTR mode encrypt the nonce and counter to the block cipher then XOR-ed with the plaintext to make the ciphertext. 

Because of how nonce and counter work (chatgpt explain this well), the first block will encrypt '475045713653717a7936644c6d654d00' (let's call this nocount, nonce + counter) then XOR it with the first 16 bytes of plaintext to get the first 16 bytes of the ciphertext, the second block's nocount is '475045713653717a7936644c6d654d01' then do the rest like the first block, the third block's nocount is '475045713653717a7936644c6d654d02', and so on

Because block cipher is the same for all AES modes, giving the nocount to the server will give us the same encrypted nocount as if it's encrypted in AES CTR before XOR-ed with the plaintext. 

Finally, concatenate all of the encrypted nocount then XOR it to the encrypted flag to get the plaintext (actual Flag)

Note : 
- dont forget to cut the AES-EBC encrypted nocount in half because of how the padding works 
- I give no script because i only write the script for the final XOR, the rest was done manually