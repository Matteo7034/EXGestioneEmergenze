# EXGestioneEmergenze

- Sistema MultiThread C11 tramite chiamate POSIX
- Emergenze raccolte tramite una coda 
- distribuzione risuorse in modo concorrente
- forze ordine: Pompieri ....
- Priorità:
- evitare situazioni di DeadLock
- Gestire la prevenzione della starvation

## Destrizione entita di sistema

- griglia rettangolare (x,y)
Es: 
Griglia (x,y):

|x/y| 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| 1 |   | x |   |   |
| 2 | x |   | x | x |
| 3 |   |   | x |   |


- Ogni unita di soccorso dispone di un sistema di localizzazzione
  rappresentata tramite  un gemmelo digitale riflette
	- tipo (esempio Pompieri)
	- stato (disponibile o in servizio)
	- posizione nella griglia
typedef enum  = Pompiere|

typedef struct Unita{
    int x;
    int y;
    int state;
    char rscuer_type_name;
}Unita;

- ogni emergenza rimossa dalla coda e caratterizzata da tipo posizione
  e istante in cui è stata registrata

## Obbiettivo

- assegganre i soccorritori appropriati a ciascuna emergenza ottimizzando
  la probabilita che le emergenze siano gestite rapidamente.

- Tipi di emergenze e soccorritori disponibili sono delineati da file di confiugrazione
  specifici:
	- rescuers.conf soccorritori
	- emergency_types.conf tipo di emergenza
	- env.conf dimensione dell'ambiente

- Si può assumere che il contenuto dei file non cambi e debba essere letto solo
  Durante l'avvio del programma

- I file devono essere letti tramite parser dedicati:
	- parse_rescuers.c
	- parse_emergency_types.c
	- parse_env.c

## Tipi e instanze di Soccorittori

Sintassi rescuers.conf
Formato BNF:

	<config> ::= <entry> | <entry> "\n" <config>
	<entry> ::= "[" <name> "]" "[" <num> "]" "[" <speed> "]" "[" <base> "]"
	<base> ::= <x_coord> ";" <y_cordd>

Ongi tipo di soccrritore ha:
	- nome
	- cardinalità int che rappresnta quanti mezzi può disporre
	- velocita espressa (caselle al secondo)
	- coordinate della loro base 
Esempio:
    [Pompieri][5][2][100;200]
    [Ambulanza][25][4][150;250]

- Strutture dati che corrispondono ai tipi e alle istanze dei soccoritori disponibili.
    - Per ogni soccrittore:
        - rescuer_type_t
        - rescuer_type_twin_t (pari al numero totale di mezzi a loro disposizione)
            - stato specifico:
            - IDLE (libero)
            - ON_ROUTE_TO_SCENE (dirigendo nella scena di emergenza.)
            - ON_SCENE ( si trova sulla scena di emergenza)
            - RETURNING_TO_BASE ( sta tornando alla base ) 
            - !!! Inizialmente assegnato a IDLE

```
typedef enum {
     IDLE , EN_ROUTE_TO_SCENE , ON_SCENE , RETURNING_TO_BASE
} rescuer_status_t ;

typedef struct {
    char * rescuer_type_name ;
    int speed ;
    int x ;
    int y ;
} rescuer_type_t ;

typedef struct {
    int id ;
    int x ;
    int y ;
    rescuer_type_t * rescuer ;
    rescuer_status_t status ;
} rescuer_digital_twin_t ;

```

## Tipi Di Emergenze

La sintassi usata per descrivere i tipi di emergenza, in formato BNF, è la seguente:
 
    <config> ::= <entry> | <entry> "\n" <config>
    <entry> ::= "[" <name> "]" "[" <priority> "]" <rescuers>
    <rescuers> ::= <rescuer> | <rescuer> <rescuers>
    <rescuer> ::= <rescuertype> ":" <number>,<time_in_secs>;

Priorita': 
- 0 bassa
- 1 media
- 2 alta

Es:


