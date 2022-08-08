W lipcu tego roku, w ramach procesu rekrutacyjnego dostałem całkiem spore zadanie do zrobienia. Ponieważ poświęciłem na to dużo swojego wolnego czasu, 
uważam że dobrym pomysłem jest umieszczenie go tutaj.

Projekt wymagał ode mnie utworzenia nowych kolumn do istniejącej tabeli danych, używając do tego informacji dostarczonych przez inny dataframe.
Wszystko co zrobiłem, umieszczone jest w pliku, w tym repozytorium.

Oprócz tego musiałem stworzyć również model danych oraz dokładnie go opisać. Wykonałem to na modelu Snowflake.
Jego graficzne przedstawienie znajduje się poniżej. 

![image](https://user-images.githubusercontent.com/93926070/183463471-372fb669-bc98-4d52-b39d-72b02f3942bc.png)

Oto opis poszczególnych tabel oraz relacji między nimi:

1. BOOKINGS
   - booking_id (INT) - prim-key, not-null, unique, auto-increment
   - booking (VARCHAR(10)) - not-null,

	Pierwsza kolumna (booking_id) jest kluczem głównym, bedącym w relacji z tabelą tickets. Każdy wiersz musi być
	unikalny, przyporządkowany do jednej rezerwacji. Dzięki autoinkrementacji, wartość kolejnego wiersza
	automatycznie zwiększa się o jeden, w porównaniu do wiersza poprzedniego.
	Booking – tabela zawierająca nazwy rezerwacji, każda utworzona rezerwacja otrzymuje swój unikatowy numer(z
	kolumny booking_id).
	Nazwa TOBER oznacza lot do Berlina(to Berlin). Liczba, oznacza numer zamówienia, na przykład danego dnia,
	czy tygodnia. Dla przykładu wartość TOLHR_843 oznacza 843 rezerwację na lot do Londynu(lotnisko London
	Heathrow Airport z oznaczeniem LHR).VARCHAR(10) służy do określenia ilości znaków możliwych do wpisania.
	W tym przypadku nie potrzeba „zapychać” pamięci większymi wymaganiami. Kolumna nie może być pusta, oraz
	musi być unikalna. Miejsce docelowe lotu może się wielokrotnie powielać, o tyle numer rezerwacji jest zawsze
	inny, ponieważ przyjmuję, że jedna tabela odpowiada konkretnemu tygodniowi czy miesiącu a liczba rezerwacji
	mieści się w podanym przedziale czasowym. ```
	
2. TICKETS
   - ticket_id (INT) - prim-key, not-null, unique, auto-increment
   - booking_id (INT) - not-null
   - ticket (VARCHAR(8)) - not-null,
   - passenger_id(INT) - not-null, unique
   - segments_id(INT) - not-null, unique
   - ancillary_id(INT) - not-null, unique
	
	Tak samo jak bilet loticzny zawiera wszystkie najważniejsze informacje dotyczące lotu, tak podobną funkcję ma
	tablea TICKETS. Jest w relacji z innymi tabelami dotyczącymi: danych pasażera(id_passengers), dodatkowych
	usług(ancillary_id), czy odcinków podróży(segments_id).
	Ticket_id to kolumna nadająca automatycznie unikalny i kolejny numer dla biletu. Nie wymaga wprowadzania
	dzięki opcji auto-inkrementacji, nie może się powielać(jeden konkretny bilet, na jedną osobę).
	Kolumna booking_id posiada tylko jedną wartość – nie może być pusta. Może się za to powielać, ponieważ
	istnieje opcja kupna kilku biletów na jedną rezerwacje. Nie jest kluczem głownym, ponieważ odnosi się do tabeli
	z rezerwacjami.
	Kolumna ticket to nazwa biletu, przeznaczyłem na to 8 znaków, ponieważ nazwa składa się z dwóch Kodów
	IATA- lotniska z którego wyruszamy, oraz docelowego(bez uwzględniania przesiadek) oddzielonych myślnikiem,
	np. „WAW-MAD”.
	Ostatnie trzy kolumny to klucze, które tworzą relację z odpowiadającymi im tabelami.

3. PASSENGERS
   - passenger_id(INT) prim-key, not-null, unique,
   - first_name (VARCHAR(20)) not-null,
   - last_name (VARCHAR(25)) not-null,
   - phone_number (VARCHAR(15)) not-null,
   - mail (VARCHAR(45)) not-null,
	
	Tabela zawiera dane wszystkich pasażerów, którzy złożyli rezerwacje. Kolumna passenger_id to jak sama
	nazwa wskazuje – numer id każdego pasażera, musi być unikalny. Jest kluczem głównym tej tabeli.
	W przypadku Kolumny phone_number nie zastosowałem zmiennej INT, ponieważ niektóre numery mogą być
	wprowadzane ze znakiem „+” na początku – który nie jest inteagerem(na przykład numer kierunkowy do polski –
	zaczynający się od +48).
	Jednak zdecydowanie lepszym rozwiązaniem było by ustalenie jednego sposobu wprowadzania numerów, na
	przykład bez znaku na początku, z numerem kierunkowym, bez spacji (48123456789)
	Na adres e-mail(kolumna mail) przeznaczyłem aż 45 znaków, ponieważ zdaję sobię sprawę, że niektóre maile
	związane z firmami czy instytucjami są długie.

4. ANCILLARY
   - ancillary_id (INT) prim-key, not-null, unique,
   - additional_baggage (VARCHAR(3)) not-null,
   - paid_place (VARCHAR(3)) not-null,
   - insurance (VARCHAR(3)) not-null,
   - paid_food (VARCHAR(3)) not-null
	
	Pierwsza kolumna, pełni dokładnie taką samą rolę jak kolumna passenger_id w poprzedniej tabeli. Jest to
	unikalny numer będący w relacji z tabelą tickets, służacy do przekazywania informacji o dodatkowych
	udogodnieniach.
	Następne cztery kolumny zawierają informację o wybraniu (wartość „YES”) bądź rezygnacji (wartość „NO”) z
	konkretnej usługi.

5. SEGMENTS
   - segment_id(INT) prim-key, not-null,
   - segment(VARCHAR(8)) not-null,
   - airport_id(INT) not-null,
	
	Tutaj sytuacja jest nieco inna. Dlatego że dla każdego biletu może być (w teorii) nieskończona ilość przesiadek,
	a każdy wiersz opisuje jeden lot bez przerwy, czy przesiadki. To znaczy że mając bilet Warszawa – Madryt w
	jedną stronę, możemy mieć w przypadku lotu z przesiadkami dwa wiersze(„WAW-BER”, oraz „BER-MAD”), dla
	których pierwsza kolumna przyjmuje tą samą wartość.
	Airport_id obsługuję relacje z tabelą zawierającą informacje o konkretnym lotnisku. Wprowadzane tutaj id
	odwołuje się do lotniska na którym czeka nas następne lądowanie.
	
6. AIRPORTS
   - airport_id(INT) prim-key, not-null, unique, auto-increment
   - airport_shortcut(VARCHAR(3)) not-null,
   - city(VARCHAR(30)) not-null,
   - country(VARCHAR(20)) not-null,
   - continent(VARCHAR(15)) not-null,
   - airport(VARCHAR(120)) not-null,
	
	airport_id to klucz, dzięki któremu możemy przyporządkować lotnisko, do lotu, nie może się dublować, każde
	lotnisko ma swój własny numer, przsypisywany automatycznie.
	airport_shortcut – kod IATA lotniska



Poniżej przedstawiam przykładowe wartości, które umieściłem w tabelach.:

![image](https://user-images.githubusercontent.com/93926070/183465174-497ea44e-06ec-4a93-8598-2b81b8291ee8.png)
![image](https://user-images.githubusercontent.com/93926070/183465269-90c141c9-dba2-4f34-b263-42fe4b09a553.png)

Za pomocą tak zaprojektowanej bazy danych, możemy pozyskiwać dowolne informacje dotyczące lotu,
rezerwacji czy biletu, na przykład:

![image](https://user-images.githubusercontent.com/93926070/183465439-68137e12-9758-48ac-8e78-19356f0f7c83.png)

Wynik: 

![image](https://user-images.githubusercontent.com/93926070/183465543-41bfc229-2d66-4eb9-a256-f0925bc76780.png)







	



























