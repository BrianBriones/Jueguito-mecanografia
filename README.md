#  Jueguito de Mecanografia

Un petit **joc de mecanografia** desenvolupat amb **Vue 3**, on hauràs d’escriure paraules tan ràpid com puguis abans que s’acabi el temps ⏱️.  
Competeix contra tu mateix o amb altres jugadors i comprova la teva velocitat i precisió al teclat!

---

##  Objectiu del joc

L’objectiu és escriure correctament totes les paraules que apareixen a la pantalla **abans que s’esgoti el temps**.  
Cada paraula completada registra el **temps que has trigat** i el **nombre d’errors** comesos.  
Al final, podràs veure un resum amb el teu **rendiment total**.

---

##  Tecnologies utilitzades

-  **Vue 3** (amb sintaxi `<script setup>`)  
-  **CSS Scoped** per a l’estil del component  
-  **Temporitzador reactiu** amb compte enrere  
-  **Estadístiques locals** de temps i errors  
-  **Llista de jugadors connectats** (preparat per multijugador)  
-  **Disseny responsive** adaptat a mòbil i tauleta  
-  **Teclat visual** que reacciona a les tecles reals que prems


##  Com jugar

### 🔹 1. Inici del joc

Quan obres el joc, el **temporitzador comença automàticament** (per defecte, 30 segons).

  **Temporitzador**  Mostra el temps restant i una barra de progrés visual. 
  **Paraula activa** És la paraula que has d’escriure actualment. 
  **Color actiu**  La paraula activa s’il·lumina en groc. 
  **Teclat visual**  Mostra les tecles que estàs prement en temps real. 


### 🔹 2. Escriure paraules


 Escriu les lletres correctament  Les lletres apareixen en **blanc**  
 Escriu una lletra incorrecta  La lletra apareix en **vermell**  i es compta com error 
 Completa tota la paraula  Es guarda el **temps trigat** i els **errors** 
 Canvi automàtic  Quan acabes una paraula, el joc passa a la **següent** 

 **Consell:** Si t’equivoques, pots esborrar amb *Backspace* i corregir-te abans de continuar.

---

### 🔹 3. Final del joc

El joc acaba quan:

- El temps arriba a **0 segons**, o  
- Has completat **totes les paraules** de la llista  

Quan això passa, apareix la pantalla de **Resultats del joc**:
