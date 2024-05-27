//Code Par : Rayen Bouafif & Bouchriha Issmail 
// Le 26/05/2024 
//Title : Algorithme Simplexe méthode tabulaire 

#include <stdio.h>
#include <stdbool.h>

#define MAX_CONSTRAINTS 10
#define MAX_VARIABLES 10

void initializeTableau(float tableau[MAX_VARIABLES][MAX_CONSTRAINTS], int numConstraints, int numVariables) {
    float coe_objective_func;
    printf("Entrer les coefficients de la fonction objective : \n");
    for (int i = 1; i <= numVariables; i++) {
        scanf("%f", &coe_objective_func);
        tableau[1][i] = coe_objective_func; // Z Premiere ligne 
    }

    // Coefficients des contraintes 
    printf("Entrer les coefficients des contraintes : \n");
    for (int i = 2; i <= numConstraints + 1; i++) {
        for (int j = 1; j <= numVariables; j++) {
            float coe_constraint_func;
            scanf("%f", &coe_constraint_func);
            tableau[i][j] = coe_constraint_func;
        }
    }

    // Initialiser les variables d'écart et les right-hand side (RHS)
    printf("Entrer RHS de votre PL : \n");
    for (int i = 2; i <= numConstraints + 1; i++) {
        for (int j = numVariables + 1; j <= numVariables + numConstraints; j++) {
            tableau[i][j] = (i == j - numVariables + 1) ? 1.0 : 0.0;
        }
        float RHS;
        scanf("%f", &RHS);
        tableau[i][numVariables + numConstraints + 1] = RHS;  // Last column
    }
    tableau[1][numVariables + numConstraints + 1] = 0;
}

void showTableau(float tableau[MAX_VARIABLES][MAX_CONSTRAINTS], int numVariables, int numConstraints) {
    // Afficher les noms des variables : x1, x2, ...
    for (int i = 1; i <= numVariables; i++) {
        printf("  x%d  ", i);
    }
    // Afficher les noms des variables d'écart : s1, s2, ...
    for (int i = 1; i <= numConstraints; i++) {
        printf("  s%d  ", i);
    }
    // Afficher la RHS
    printf("%6s\n", "RHS");

    // Affichage des valeurs de tableau
    for (int i = 1; i <= numConstraints + 1; i++) {
        for (int j = 1; j <= numConstraints + numVariables + 1; j++) {
            printf("%6.1f", tableau[i][j]);
        }
        printf("\n");
    }
    printf("---------------------------------------------\n");
}

//Verifier que Z est négatif
bool checkAllNumberInZRow(float tableau[MAX_VARIABLES][MAX_CONSTRAINTS], int numVariables, int numConstraints ) {
    for (int i = 1; i <= numVariables + numConstraints; i++) {
        float temp = tableau[1][i];
        if (temp > 0.000000){
                return false;
        }
    }
    return true;
}

//Chercher la plus grande valeur de z (the pivot column)
int BiggestNumberInFirstRowIndex(float arr[MAX_VARIABLES], int numVariables, int numConstraints) {
    float BiggestNumber = arr[1];
    int index = 1;
    for (int i = 1; i <= numVariables + numConstraints; i++) {
        if (arr[i] > BiggestNumber) {
            BiggestNumber = arr[i];
            index = i;
        }
    }
    return index; 
}

//Calcule des indicateurs et chercher la plus petit indicateur <0
int smallestIndicatorIndex(float tableau[MAX_VARIABLES][MAX_CONSTRAINTS], int numVariables, int numConstraints, int pivot_col_index ) {
    float indicator[MAX_CONSTRAINTS];

    for (int i = 2; i <= numConstraints+1; i++ ) {  //indicator = RHS / valeur correpondante en colonne pivot
        indicator[i] = (float)tableau[i][numConstraints+numVariables+1] / tableau[i][pivot_col_index];
    }
    
    //Chercher l'indicateur le plus petit 
    float smallestIndicator = 10000000.0;
    int index = 1;
    for (int i = 2; i <= numConstraints+1; i++) {
        if (indicator[i] > 0.000000 && smallestIndicator > indicator[i] ) {
            smallestIndicator = indicator[i];
            index = i;
        }
    }
    return index; 
}

//Mise à jour 
void makeNewTableau(float tableau[MAX_VARIABLES][MAX_CONSTRAINTS], int numVariables, int numConstraints, int pivot_row_index, int pivot_col_index, float pivot_variable) {
   for (int i = 1; i <= numVariables+numConstraints+1; i++) {
        tableau[pivot_row_index][i] = (float)tableau[pivot_row_index][i] / pivot_variable;
   }

    for (int i = 1; i <= numConstraints+1; i++) {
        if (i != pivot_row_index) { //sauf la ligne pivot 
            
            // Mise à jour des lignes 
            // Ici on veut Annuler tous la colonne pivot sauf le ligne correpondant au pivot 
            float temp = tableau[i][pivot_col_index] * (-1);
            for (int j = 1; j <= numVariables+numConstraints + 1; j++) {
                tableau[i][j] = tableau[i][j] + temp * tableau[pivot_row_index][j];
            }

        }
    }
}

int checkBasic(float arr[MAX_CONSTRAINTS], int numConstraints) {
    int count = 0;
    int index = -1;
    for (int i = 1; i <= numConstraints+1; i++) {
        if (arr[i] != 1 && arr[i] != 0) {
            return -1;   
        }
        else {
            if (arr[i] == 1) {
                count++;
                index = i;
                
            }
        }
    }
    
    if (count == 1) {
        return index;
    }
    else { 
        return -1;
    }
}

void showSolution(float tableau[MAX_VARIABLES][MAX_CONSTRAINTS], int numVariables, int numConstraints) {
    
    printf("Optimal solution z max equal to %.1f at: ", (-1)*tableau[1][numConstraints+numVariables+1]);
    for (int i = 1; i <= numVariables; i++) {
        float x_column[MAX_CONSTRAINTS];  //Transposé 
        for (int j = 1; j <= numConstraints+1; j++) {
            x_column[j] = tableau[j][i];
        }
       
        int index = checkBasic(x_column,numConstraints);
       
        if (index != -1) {
            printf("x%d = %.1f, ",i,tableau[index][numConstraints+numVariables+1]);
        } else {
            printf("x%d = 0.0, ",i);
        }
    }
    printf("\n");
}

int main() {
    float tableau[MAX_VARIABLES][MAX_CONSTRAINTS] = {0};
    int numVariables, numConstraints;

    printf("Enter number of variables: \n");
    scanf("%d", &numVariables);
    printf("Enter number of constraints: \n");
    scanf("%d", &numConstraints);

    initializeTableau(tableau, numConstraints, numVariables);
    printf("Original tableau\n");
    showTableau(tableau, numVariables, numConstraints);
    printf("\n");

    while (!checkAllNumberInZRow(tableau, numVariables, numConstraints)) {
        int pivot_col_index = BiggestNumberInFirstRowIndex(tableau[1], numVariables, numConstraints);
        int pivot_row_index = smallestIndicatorIndex(tableau, numVariables, numConstraints, pivot_col_index);
        float pivot_variable = tableau[pivot_row_index][pivot_col_index];
        makeNewTableau(tableau, numVariables, numConstraints, pivot_row_index, pivot_col_index, pivot_variable);
        showTableau(tableau, numVariables, numConstraints);
    }

    showSolution(tableau, numVariables, numConstraints);

    return 0;
}