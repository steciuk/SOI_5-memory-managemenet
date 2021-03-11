# SOI_5-worst-fit
Memory managemenet - worst fit algorithm in Minix
## ROZWIĄZANIE PROBLEMU
Zaimplementowałem algorytm WORST_FIT, umożliwiając użytkownikowi możliwość zmiany algorytmu alokacji pamięci oraz stworzyłem dwa dodatkowe wywołania systemowe wywołujące funkcje:  
do_hole_map – zgdonie z poleceniem zwracające bufor wypełniony adresami i wielkościami wolnych w pamięci miejsc  
do_worst_fit -  zgodnie z poleceniem, umożlwiająca użytkownikowi zmianę domyślnego algorytmu alokacji pamięci  
minix_usr/src/mm/alloc.c   (linie 27 – 33, 53 – 86, 101, 104 – 105, 116 – 117, 124 – 198)  
1.	 Zdefiniowanie 4 możliwych wersji algorytmu: FIRST, WORST, FIRSTOUT, WORSTOUT przy czym wersja 3 i 4 różnią się od zadanych w poleceniu (1 i 2) tym, że dodatkowo na bieżąco wypisują na ekran zajmowane bloki pamięci.
2.	Zdefiniowanie zmiennej przechowywującej obecnie wybrany algorytm
3.	Zdefiniowanie zgodnych z poleceniem funkcji do_hole_map i do_worst_fit
4.	Zaimplementowanie algorytmu WORST_FIT w funkcji alloc_mem(), który zamiast wybierać pierwszy odpowiednio duży znaleziony blok pamięci, wybiera największy możliwy i ucina z niego określony poleceniem chmem kawałek pamięci.
(zgodnie z dokumentacją (man chmem) dla programu tak naprawdę alokowany jest delikatnie więcej pamięci niż określone przez chmem, w tym przypadku zamiast 8Kb jest zajmowane 9Kb)

Tworzenie wywołań systemowych:  
minix_usr/include/minix/callnr.h (linie 1, 68, 69)  
minix_usr/src/fs/table.c (linie 78, 79)  
minix_usr/src/mm/proto.h (linie 14, 15)  
minix_usr/src/mm/table.c (linie 96, 97)  
 
## TESTY I INTERPRETACJA WYNIKÓW
Testy zostały zrealizowane za pomocą udostępnionych na stronie prowadzącego programów i skryptów:
1.	x.c - symulującego program testowy realizujący obliczenia
2.	t.c - zasadniczo wywołujący funkcję do_hole_map()
3.	w.c - przyjmujący argumenty od 0 do 3 umożlwiający przełączanie używanego algorytmu między utworzonymi przeze mnie 4 wersjami

Testy uruchamiane były skryptem również znajdującym się na stronie prowadzącego, zmodyfikowanym jedynie po to by wyświetlane były jeszcze wolne bloki pamięci przed pierwszą alokacją dla programów testowych (pierwsza linijka) oraz podzielonym osobno na test first fit i worst fit.

First fit:  
![Obraz1](https://user-images.githubusercontent.com/48189079/110821599-84368880-8290-11eb-9e93-2c69c8c01996.png)  
Działanie programu przebiega na pierwszy rzut oka nie zgodnie z przewidywaniami. Przed każdą z 10 pierwszych linijek (poczynając od drugiej) alokowane powinno być 9Kb z pierwszej wolnej dziury. A później zwalniana powinna być ta pamięć w odwrotnej kolejności. Widzimy, jednak, że mimo, że pierwszy blok pamięci ma rozmiar 68, to pamięć dla naszego programu testowego alokowana jest z kolejnych bloków. Aby zbadać ten problem stworzyłem napomniane wyżej dwie dodatkowe wersje algorytmu alokującego: FIRSTOUT i WORSTOUT. Algorytmy te na bieżąco wypisują zajmowane bloki pamięci. 
![Obraz2](https://user-images.githubusercontent.com/48189079/110821650-9284a480-8290-11eb-8e29-7e0cee8c092f.png)  
Na zrzucie ekranu wyraźnie widać, że przed alokacją 9Kb dla naszego procesu testowego, następuje alokacja dla procesów na 62Kb sprawiając, że w momencie wywoływania naszego programu testowego – pierwszy blok pamięci wcale nie ma wielkości 68Kb jakby wskazywały na to logi, a 6Kb co uniemożliwia wykorzystanie tej dziury. Alokowanie tych 62Kb wynika z implementacji sposobu tworzenia nowych procesów w systemach unixowych. Jest to pamięć wykorzystywana przez fork i exec.
Wart uwagi jest również fakt, że zgodnie z przewidywaniami, przed rozpoczęciem testów i po ich zakończeniu, stan pamięci jest taki sam.

Worst fit:  
![Obraz3](https://user-images.githubusercontent.com/48189079/110821748-a92afb80-8290-11eb-9471-e1bdfaa10748.png)  
Worst fit alokuje pamięć w największym znalezionym kawałku pamięci (tutaj będzie to zawsze ostatnia dziura). Dzieje się tak i co alokację, największy segment zmniejszany jest o kolejne obszary pamięci. Komentarza wymagają jednak nowo pojawiające się dziury wielkości 62Kb.
Jest to konsekwencją tego samego “problemu” z jakim mieliśmy do czynienia w przypadku first fit. Tu jednak rezultat jest nieco inny.
Z największego obszaru alokowane jest 62Kb na fork i exec, następnie 9Kb na nasz program testowy. Zwalniane jest 62Kb od procesów tworzących, w rezultacie czego powstaje dziura. Nie zostanie ona jednak wykorzystana przy kolejnym wywołaniu procesu testowego, bo pamięć zawsze ma być przydzielana z największego bloku pamięci, więc znów zaalokowane zostanie 62Kb, potem 9Kb, znów zwolnione 62Kb i tak w kółko tworząc kolejne dziury. W momencie, gdy instancje naszego programu testowego będą kończyć swoje działanie (po 11 linijce), ich pamięć będzie stopniowo zwalniana a 62Kb’owe dziury będą łączone w coraz to większe bloki pamięci. Ostatecznie dołączone zostaną to największego bloku - przywracając stan pamięci sprzed rozpoczęcia testów.


