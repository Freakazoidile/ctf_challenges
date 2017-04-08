<h2>Challenge</h2>

There was a IP address and port to netcat into. The challenge then presented it's self. A list of number and associated keywords. Pressing enter the challenge would begin. A number was then printed. After playing with the challenge the correct response would be:
* If the number given was a number in the list enter in the keyword associated with it.
* If the number was not in the list, but was divisible by numbers in the list enter in order the keyword(s) without spaces 
* If the number was not in the list and not divisible, enter "FAKE NUMBER"


<h2>Solution / Script</h2>

I created a script that connected and hit "enter" to begin. Then added onto to reading in the number is received and checking if it was in the list the challenge provided (which I put into a python dictionary). Then added `%` modulo to determine if it was divisible and if so to concatenate the keyword onto the result. After everything was checked it would return the answer. If all numbers were checked and not in the dictionary, or not divisible by a number in the dictionary it returned "FAKE NUMBER".  This script took ~2-5 minutes to run then it printed out the final response, the flag. 

```
sun{I_g1vE_u_nUM3r0_u_G1v3_m3_alt3Rn4TE_nUMEr0}
```

```python
import socket
import time

dict = {"2":"fallout", "3":"survivor", "5":"comrade", "7":"nuclear", "11":"apocalypse", "13":"shelter", "17":"war", "19":"radioactive", "23":"atom", "29":"bomb", "31":"radiation", "37":"destruction", "41":"mushroom", "43":"armageddon", "47":"disaster", "53":"pollution", "59":"military", "61":"science", "67":"winter", "71":"death", "73":"atmosphere", "79":"bunker", "83":"soldier", "89":"danger", "97":"doomsday"}

def calc(data):
  result = ""
  for key in sorted(dict.iterkeys(), key=int):
    if data == key:
      result = dict[data]
      break
    else:
      divisible = int(data)%int(key)
#      print "data%key="+data+key+"="+divisible
      if divisible == 0:
        result += dict[key]

  if  result == "":
    result = "FAKE NUMBER"
  return result

s = socket.socket()

s.connect(("34.208.132.18",30001))
time.sleep(1)
ans = s.recv(2048)
print ans


s.send("\n")
while 1:
  time.sleep(2)
  ans = s.recv(1024).strip()
  
  if ans == "":
      break
  print "got: ", ans
  result=calc(ans)
  print result
  s.send(str(result)+"\n")
```

