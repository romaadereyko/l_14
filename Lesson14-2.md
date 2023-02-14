# Hole file
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt
Mike Harrington:(510) 548-1278:250:100:175
Christian Dobbins:(408) 538-2358:155:90:201
Susan Dalsass:(206) 654-6279:250:60:50
Archie McNichol:(206) 548-1348:250:100:175
Jody Savage:(206) 548-1278:15:188:150
Guy Quigley:(916) 343-6410:250:100:175
Dan Savage:(406) 298-7744:450:300:275
Nancy McNeil:(206) 548-1278:250:80:75
John Goldenrod:(916) 348-4278:250:100:175
Chet Main:(510) 548-5258:50:95:135
Tom Savage:(408) 926-3456:250:168:200
Elizabeth Stachelin:(916) 440-1763:175:75:300
```
# All numbers
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | cut -d ':' -f 2
(510) 548-1278
(408) 538-2358
(206) 654-6279
(206) 548-1348
(206) 548-1278
(916) 343-6410
(406) 298-7744
(206) 548-1278
(916) 348-4278
(510) 548-5258
(408) 926-3456
(916) 440-1763
```
# Dan's number
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | grep Dan
Dan Savage:(406) 298-7744:450:300:275
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | grep Dan | cut -d ':' -f 2
(406) 298-7744
```
# Susan's info
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | grep Susan | cut -d ':' -f 1,2
Susan Dalsass:(206) 654-6279
```

# All last names which begins from D
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | awk {'print $2'} | cut -d : -f 1 | grep ^D
Dobbins
Dalsass
```

# All names which begins from C or E
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | awk {'print $1'} | grep '[CE]'
Christian
Chet
Elizabeth
```

# All names which consists of 4 symbols
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | awk 'length($1) == 4 {print $1}'
Mike
Jody
John
Chet
```

# Worker's names which have 916 prefix
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | grep '(916)' | awk {'print $1'}
Guy
John
Elizabeth
```

# Mike's Deposits
```sh
root@TEST1:/srv/git/DOS013-onl/romaadereyko/Lesson14# cat vklad.txt | grep Mike | awk -F: {'print "$"$3,"$"$4,"$"$5'}
$250 $100 $175
```
