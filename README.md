# Projet-SAS
[rendu.pdf](https://github.com/bdj-02/Projet-SAS/files/8793238/rendu.pdf)


                                             /***********************/
                                             /***** Projet SAS ******/
                                            /***********************/

/*importation du fichier Données_devoir_SAS sous sas*/
filename REFFILE '/home/u60864178/projet';
filename REFFILE '/home/u60864178/projet/Données_devoir_SAS.csv';


                           /******************** PARTIE A   ****************/

                              /**************** QUESTION 1 *************/
PROC IMPORT DATAFILE=REFFILE 
	DBMS=CSV /*extension du fichier importé, ici CSV */
	OUT=WORK.MARKETING; /* nom de la table*/
	GETNAMES=YES; /*garder le nom des colonnes */
	DELIMITER=';'; /*spécifier le délimiteur qui sépare les colonnes de données dans le fichier d'entrée*/
RUN;

/* procédure permettant de voir le contenu d'une table */
PROC CONTENTS DATA=WORK.MARKETING
	varnum /* afficher les variables par ordre d'apparition de la table*/
	short; /* afficher seulement le nom des variables */
RUN;

/* Afficher le contenu d'une table */
proc print data=WORK.MARKETING;
run;

                            /*************** QUESTION 2 *************/
data WORK.MARKETING;
	set WORK.MARKETING; /* reprendre notre table marketing*/
	rename annee_naissance=date_naiss ID=id_client date_ancienneté=date_anc; /* Spécifier les colonnes qu'on veut renommer*/
run;

proc print data=WORK.MARKETING;
run;

                               /*************** QUESTION 3 *************/
PROC SQL;
	select distinct * /* Pour chaque colonne de notre table, on affiche les différentes valeurs distinctes, afin de vérifier ainsi l'absence de doublons*/
	from WORK.MARKETING;
quit;
	/* 2em Methode doublons et valeurs maquantes*/
PROC SORT DATA=WORK.MARKETING
	nodupkey;
	by _all_;
RUN;
proc print data=WORK.MARKETING;
run;

proc sql;
	describe table WORK.MARKETING; /* afficher la description de notre table*/
run;

/** 2) Vérification des valeurs manquantes de tous les varaibles étudiées **/
Proc means data = WORK.MARKETING n nmiss;
     var _numeric_;
run;
Proc freq data =WORK.MARKETING;
    tables _character_ / missing;
run;
/*************** QUESTION 4 *************/

/* Après verification suite au résultat de la question précédente */
/* On constate donc ques toutes le type des variables de notre    */ 
/* table est bien le même que celui spécifié dans le tableau 1    */

/* Dans le cas où il y aurait une ou plusieurs varaibles qui dont */
/* le type ne serait pas celui spécifié dans le tableau 1 on peut */
          /* donc excécuter la procédure si dessous          */
          
  /*proc sql;
     select name into:liste separated by ';' from WORK.MARKETING;
     quit;*/


                               /*************** QUESTION 5 *************/
data work.marketing;
	set work.marketing;
	drop MT_OR; /* Suppression de la variable mt_or avec la fonction drop*/
run;

                               /*************** QUESTION 6 *************/
data work.marketing;
    set work.marketing;
    format reponse_2 $40.; /* on déclare notre nouvelle variable en initialisant sa taille et son type (ici chaîne de caractère de taille 40 max)*/
    length reponse_2 $40 default=4. ;

    if reponse=1 then reponse_2='Le client a réalisé un achat'; 
    else reponse_2='Le client n’a pas réalisé un achat';
run;

proc print data=work.marketing;
run;


                           /******************** PARTIE B   ****************/

                              /**************** QUESTION 7 *************/
PROC GCHART DATA = WORK.MARKETING;
    vbar  Niveau_educatif Situation_maritale reponse_2 ;
run; quit;

                              /**************** QUESTION 8 *************/
PROC FREQ DATA = work.marketing;
    table Situation_maritale*reponse_2 Niveau_educatif*reponse_2 / nopercent norow nocol;
run;

/************************************************* CONSTAT ***************************************/

/* La majorité des clients sont mariés et diplomés                         */
/* Nous constatons que la majorité des personnes n'ayant pas réalisé d'achat sont mariées
/* La plupart des clients ayant réalisé un achat sont célibataire et diplomés
*  Le nomnbre total de clients n'ayant pas réalisé d'achats est plus élevé que le nombre de clients ayant réalisés un achat */


  /**************** QUESTION 9 *************/

proc means data=work.marketing;
	var nb_achat_web nb_achat_catalogue nb_achat_magasin revenu;
run;


proc sort data=work.marketing; by plainte; run;

proc means data=work.marketing;
	var mt_fruit mt_viande mt_poisson mt_bonbon mt_vin;
	by plainte;
run;

proc sort data=work.marketing; by reponse; run;

proc GCHART data=work.marketing;
	pie reponse;
run;


proc sql;
	select sum(offre_accepte_1) "offre_1", sum(offre_accepte_2) "offre_2", sum(offre_accepte_3) "offre_3", sum(offre_accepte_4) "offre_4", sum(offre_accepte_5) "offre_5" from work.marketing;
run;

proc sql;
	select nb_ado, sum(mt_vin) "vin", sum(mt_fruit) "fruit", sum(mt_viande) "viande", sum(mt_poisson) "poisson", sum(mt_bonbon) "bonbon"  from work.marketing
	group by nb_ado;
quit;


proc corr data=work.marketing;
	var nb_achat_magasin nb_achat_web;
run;

                                          
 /**************** CONSTAT *************/
                             
/* Nous avons une vraie corrélation entre le nimbre d'achats réalisés en ligne et en magasin */
/* avec un coeff de corrélation == 0.50271 et une estimation de  0001  

	En moyenne le revenu des clients est de 52247
	Il y'a en moyennne 4 commandes passées en ligne et 5 en magasin et 2 via le catalogue
	
	Le produit ayant reçu plus de plaintes est le vin et celui ayant reçu moins est le poisson.

	Le nombre total de personnes ayant pas accepté l'offre à la dernière campagne est de 1906 contre seulement 334 pour ceux qui ont accepté

	et l'offre ayant reçu plus de réponses positives et l'offre 4 et la moins acceptée est l'offre 2 (seulement 30)
	
	La majorité des clients qui achètent du poisson, fruit, bonbon, viande ont aucun adolescent

*/

proc corr data=work.marketing;
	var nb_achat_web nb_web_visite nb_achat_catalogue;
run;


/**************** QUESTION 10 *************/

proc format;
value vin_price low-<10='moins de 10 €'	
				10-50='entre 10 et 50 €'
				50-100='entre 50 et 100 €'
				100-150='entre 100 et 150 €'
				150-200='entre 150 et 200 €'
				200-250='entre 200 et 250 €'
				250-300='entre 250 et 300 €'
				300-350='entre 300 et 350 €'
				350-500='entre 350 et 400 €'
				500<-hight='plus de 500 €';			
value fruit_price 	low-<10='moins de 10 €'	
					10-50='entre 10 et 50 €'
					50-100='entre 50 et 100 €'
					100-150='entre 100 et 150 €'
					150-200='entre 150 et 200 €'
					200-250='entre 200 et 250 €'
					250-300='entre 250 et 300 €'
					300-350='entre 300 et 350 €'
					350-400='entre 350 et 400 €'
					500<-hight='plus de 500 €';
value viande_price 	low-<10='moins de 10 €'	
					10-50='entre 10 et 50 €'
					50-100='entre 50 et 100 €'
					100-150='entre 100 et 150 €'
					150-200='entre 150 et 200 €'
					200-250='entre 200 et 250 €'
					250-300='entre 250 et 300 €'
					300-350='entre 300 et 350 €'
					350-400='entre 350 et 400 €'
					500<-hight='plus de 500 €';
value poisson_price low-<10='moins de 10 €'	
					10-50='entre 10 et 50 €'
					50-100='entre 50 et 100 €'
					100-150='entre 100 et 150 €'
					150-200='entre 150 et 200 €'
					200-250='entre 200 et 250 €'
					250-300='entre 250 et 300 €'
					300-350='entre 300 et 350 €'
					350-400='entre 350 et 400 €'
					500<-hight='plus de 500 €';
value bonbon_price 	low-<10='moins de 10 €'	
					10-50='entre 10 et 50 €'
					50-100='entre 50 et 100 €'
					100-150='entre 100 et 150 €'
					150-200='entre 150 et 200 €'
					200-250='entre 200 et 250 €'
					250-300='entre 250 et 300 €'
					300-350='entre 300 et 350 €'
					350-400='entre 350 et 400 €'
					500<-hight='plus de 500 €';
run;


data work.marketing;
	set work.marketing;
	format prix_vin $20. prix_fruit $20. prix_viande $20. prix_poisson $20. prix_bonbon $20.;
	prix_vin=put(mt_vin, vin_price.);
	prix_fruit=put(mt_fruit, fruit_price.);
	prix_viande=put(mt_viande, viande_price.);
	prix_poisson=put(mt_poisson, poisson_price.);
	prix_bonbon=put(mt_bonbon, bonbon_price.);
run;


                              /**************** Partie C   **************/

                              /**************** QUESTION 11 *************/	
proc logistic data= work.marketing plots=roc;
	model Reponse = id_client date_naiss  
	      Revenu Nb_enfant Nb_ado date_anc NB_jours_achat mt_vin 
	      mt_fruit mt_viande mt_poisson mt_bonbon nb_achat_discount 
	      nb_achat_web nb_achat_catalogue nb_achat_magasin nb_web_visite 
	      offre_accepte_1 offre_accepte_2 offre_accepte_3 offre_accepte_4 
	      offre_accepte_5 Plainte;
run;	

/* Les variables significatives de la regression logistique *:/
/* sont les suivantes :Nb_ado, date_anc, NB_jours_achat, mt_viande, nb_achat_magazin, 
/* offre_accepte_1, offre_accepte_3, offre_accepte_4, offre_accepte_5 */
/* Au total on a conjecturé 9 variables significatives */

                             /**************** QUESTION 12 *************/

proc logistic data= work.marketing plots=ROC;
	model Reponse = id_client date_naiss  
	      Revenu Nb_enfant Nb_ado date_anc NB_jours_achat mt_vin 
	      mt_fruit mt_viande mt_poisson mt_bonbon nb_achat_discount 
	      nb_achat_web nb_achat_catalogue nb_achat_magasin nb_web_visite 
	      offre_accepte_1 offre_accepte_2 offre_accepte_3 offre_accepte_4 
	      offre_accepte_5 Plainte / SELECTION=BACKWARD;  
run;
/* Les variables que le modèle conserve sont tout simplement les variables signifiactives */
/* ennocées lors de la première regression logistique sans spécification des méthodes backward, forward ou stepwise */
/* ajouté à cela de nouvelles variables suivantes : nb_achat_discount, nb_achat_web et enfin offre_accepte_2 */
/* Au total on a 12 varaibles conservées par le modèle */


                             /**************** QUESTION 13 *************/
proc logistic data= work.marketing plots=ROC;
	model Reponse = id_client date_naiss  
	      Revenu Nb_enfant Nb_ado date_anc NB_jours_achat mt_vin 
	      mt_fruit mt_viande mt_poisson mt_bonbon nb_achat_discount 
	      nb_achat_web nb_achat_catalogue nb_achat_magasin nb_web_visite 
	      offre_accepte_1 offre_accepte_2 offre_accepte_3 offre_accepte_4 
	      offre_accepte_5 Plainte / SELECTION=STEPWISE;  
run;

/* Les variables sont conservées sont les même que celui du modèle */
/* avec la SELECTION=BACKWARD et la selection SELECTION=FORWARD    */

                            /**************** QUESTION 14 *************/
%macro Proc_log(BACKWARD, FORWARD, STEPWISE);
	proc logistic data= work.marketing plots=ROC;
	model Reponse = id_client date_naiss  
	      Revenu Nb_enfant Nb_ado date_anc NB_jours_achat mt_vin 
	      mt_fruit mt_viande mt_poisson mt_bonbon nb_achat_discount 
	      nb_achat_web nb_achat_catalogue nb_achat_magasin nb_web_visite 
	      offre_accepte_1 offre_accepte_2 offre_accepte_3 offre_accepte_4 
	      offre_accepte_5 Plainte / SELECTION=FORWARD SELECTION=BACKWARD SELECTION=STEPWISE;  
	run;
	quit;
	
%mend;
%Proc_log(BACKWARD, FORWARD, STEPWISE);

                             /**************** QUESTION 15 *************/
proc logistic data= work.marketing plots=ROC;
	OUTPUT OUT=SORTIE PREDPROBS = I;
	model Reponse = id_client date_naiss  
	      Revenu Nb_enfant Nb_ado date_anc NB_jours_achat mt_vin 
	      mt_fruit mt_viande mt_poisson mt_bonbon nb_achat_discount 
	      nb_achat_web nb_achat_catalogue nb_achat_magasin nb_web_visite 
	      offre_accepte_1 offre_accepte_2 offre_accepte_3 offre_accepte_4 
	      offre_accepte_5 Plainte / SELECTION=STEPWISE;  
run;
Proc Print data=work.marketing;
RUN;

                             /**************** QUESTION 16 *************/
proc freq data=work.sortie;
	table _FROM_*_INTO_;
run;
                             /**************** QUESTION 17 *************/
Proc Export data = Work.marketing
	dbms=xls
	Outfile='/Projet_SAS/Données_devoir_SAS.csv';
	Delimiter=";";
Run;
