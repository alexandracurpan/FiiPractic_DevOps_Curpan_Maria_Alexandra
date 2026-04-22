# Structura proiectului:

- Infrastructura: am utilizat Vagrantfile pentru a crea doua masini virtuale conectate printr-o retea de tip Host-Only Network, pentru ca cele doua vm-uri sa poata comunica intre ele, dar in acelasi timp sa fie izolate de restul retelei publice, astfel fiind creat un mediu sigur de testare. 
  Masina virtuala Gitlab ruleaza serverul GitLab, registry-ul care retine imaginile Docker si runner-ul care executa automatizarile, VM-ul App reprezinta server-ul de productie, unde aplicatia finala ruleaza sub forma de container. 
  Pentru ca cele doua masini sa comunice in mod automat, fara intreruperi, am configurat chei SSH intre ele. Astfel, Ansible nu se opreste de fiecare data pentru a cere parola.

- Automatizare: pentru a nu instala totul de mana pe fiecare masina, fapt care ar fi dus la posibile erori, am folosit Ansible:
  - inventory.ini care defineste cele doua masini si adresele lor IP, pentru ca Ansible sa stie unde sa execute comenzile
  - Common.yaml : playbook de baza, care asigura ca ambele masini sunt la fel, creand un mediu de rulare identic.
  - Deploy.yaml : asigura functionarea corecta a aplicatiei, descarcand ultima imagine din registry, stegand-o pe cea veche, pentru a nu exista erori, si pornirea      noului container
- CI/CD: 
  - Dockerfile : compileaza aplicatia springboot intr-un fisier si creeaza o imagine usoara, care poate fi rulata oriunde, pentru a evita erorile in care anumite       tool-uri merg pe un calculator, dar nu si pe altul
  - .gitlab-ci.yml, care defineste pipeline-ul cu doua etape: build (creeaza imaginea si o trimite in registry) si deploy (apeleaza deploy.yaml si pentru a instala     acea imagine pe masina App) 
- Web-server
  - Nginx.conf: gestioneaza traficul din https, se ocupa de securitatea lui si redirectioneaza traficul catre o pagina de mentenanta, daca ceva nu merge (de            exemplu, atunci cand netdata nu este disponibil)

# Dificultati intampinate:

   Am avut niste probleme cu laptop-ul, erau momente in care rularea simultana a celor doua masini virtuale si a unui browser sau descarcarea unor tool-uri pe vms ducea la epuizarea memoriei RAM. De exemplu, la a patra sesiune, playbook-urile Ansible care instala Docker pe ambele masini esuau din cauza timeout-urilor. Am rezolvat prin terminarea proceselor inutile, iar prin aceasta experienta m-am invatat sa salvez procesul pe virtualbox prin snapshots, pentru a putea reveni la o etapa stabila atunci cand se iveau probleme de acest tip.
   
   In configurarea propriu-zisa a masinilor, am intalnit cateva probleme, iar in rezolvarea lor, mai intai verificam daca s-a discutat despre rezolvarea lor ce canalul de help, iar daca nu am apelat in general ori la documentatiile de pe internet, ori la mentori in timpul sesiunilor. De exemplu, cand incercam sa rulez Dockerfile, imaginea openjdk:11 a generat eroare la build (“failed to get metadata”), iar cautand pe internet, am aflat ca s-ar putea sa fie din cauza ca imaginile openjdk sunt “deprecated” si nu mai primesc actualizari sau suport pentru noile versiuni de Docker. De aceea am trecut la eclipse-temurin:11-jdk. 
   
   Am avut si niste probleme cu certificatul CA: runner-ul nu se inregistra, iar browser-ul il marca “Not Secure”. Pentru rezolvarea acestei probleme am apelat la AI, pentru ca eram pierduta si nu gaseam informatii pe altundeva. Solutia a constat in a adauga SAN la certificat pentru a valida domeniul, deoarece standardele moderne de securitate nu mai accepta certificate care folosesc doar campul Common Name.
   
   De asemenea, ori de cate ori intampinam probleme in timpul sesiunilor, nu ezitam sa intreb mentorii. De exemplu, in sesiunea 3, cand nu intelegeam de ce imi dadea eroarea ca nu gasea fisierul atunci cand incercam sa accesez site-ul, iar din explicatiile unui mentor mi-am dat seama de greseala: in loc sa folosesc calea absoluta /var/www/example, creasem directorul folosind o cale relativa var/www/example, iar din acest motiv, nginx nu il putea gasi.
   

# Imbunatatiri viitoare:

   Desi proiectul este functional in etapa actuala, in viitor as vrea sa mai lucrez la configurarea unui load balancer, pentru a distribui traficul catre mai multe instante ale aplicatiei si sa folosesc alte tool-uri pentru criptarea datelor sensibile, deoarece acum parolele si token-urile sunt stocate in variabile simple de CI/CD.
