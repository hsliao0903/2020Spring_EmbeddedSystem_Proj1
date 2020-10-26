//
//  CDS 4630/CDA 5636 Embedded System Project 1
//  NAME: Hsiang-Yuan Liao
//  UFID: 43535341
//  Psim.c
//  
//  
//  Created by Hsiang-Yuan Liao on 2/7/20.
//  On my honor, I have neither given nor received unauthorized aid on this assignment.

#include <stdio.h>
#include <string.h>
#define MAX 16

//<Opcode>, <Destination Register>, <First Source Operand>, <Second Source Operand>
struct instructionNode {
    char opcode[3];
    int des_reg;
    int fst_reg;
    int sec_reg;
};

//Maintain Global Structure
struct instructionNode insNode[MAX];
int Register[MAX];
int DataMem[MAX];
int total_ins, current_ins;  //total input instructions
struct instructionNode INBnode[MAX];
int totalINB, currentINB;
struct instructionNode SIBnode[MAX];
int totalSIB, currentSIB;
struct instructionNode AIBnode[MAX];
int totalAIB, currentAIB;
struct instructionNode PRBnode[MAX];
int totalPRB, currentPRB;
int ADBnode[MAX][2]; //<des-register, address>
int totalADB, currentADB;
int REBnode[MAX][2]; //<des-reg, value>
int totalREB, currentREB;


//for get input functions
void get_instructions();
void get_registers();
void get_datamemory();
void processINB();
void processSIB();
void processAIB();
void processPRB();
void processADB();
void processDAM();
void processREB();
void processRGF();
void printINM(FILE *fptr);
void printRegister(FILE *fptr);
void printDAM(FILE *fptr);
void printINB(FILE *fptr);
void printSIB(FILE *fptr);
void printAIB(FILE *fptr);
void printPRB(FILE *fptr);
void printADB(FILE *fptr);
void printREB(FILE *fptr);

/*void print_insNode(int step) {

    printf("STEP %d:\n", step);
    
    printf("INM:");
    printINM();
    
    printf("INB:");
    printINB();
    
    printf("AIB:");
    printAIB();
    
    printf("SIB:");
    printSIB();
    
    printf("PRB:");
    printPRB();
    
    printf("ADB:");
    printADB();
    
    printf("REB:");
    printREB();
    
    printf("RGF:");
    printRegister();
    
    printf("DAM:");
    printDAM();
    
    printf("\n");
}*/

void file_print_insNode(FILE *fptr, int step) {

    if(total_ins == 0) return;
    
    fprintf(fptr, "STEP %d:\n", step);
    
    fprintf(fptr, "INM:");
    printINM(fptr);
    
    fprintf(fptr, "INB:");
    printINB(fptr);
    
    fprintf(fptr, "AIB:");
    printAIB(fptr);
    
    fprintf(fptr, "SIB:");
    printSIB(fptr);
    
    fprintf(fptr, "PRB:");
    printPRB(fptr);
    
    fprintf(fptr, "ADB:");
    printADB(fptr);
    
    fprintf(fptr, "REB:");
    printREB(fptr);
    
    fprintf(fptr, "RGF:");
    printRegister(fptr);
    
    fprintf(fptr, "DAM:");
    printDAM(fptr);
    
    fprintf(fptr, "\n");
}

void initialization(){
    int i;
    for(i=0 ; i<MAX ; i++){
        Register[i] = -1;
        DataMem[i] = -1;
    }
    
    totalINB = 0;
    currentINB = -1;
    totalSIB = 0;
    currentSIB = -1;
    totalAIB = 0;
    currentAIB = -1;
    totalPRB = 0;
    currentPRB = -1;
    totalADB = 0;
    currentADB = -1;
    totalREB = 0;
    currentREB = -1;
    
    
}

int Done(){
    
    if(total_ins == 0) return 1;
    
    if(current_ins>=total_ins && currentINB>=totalINB && currentSIB>=totalSIB && currentAIB>=totalAIB && currentPRB>=totalPRB && currentADB>=totalADB
       && currentREB>=totalREB)
        return 1;
    else
        return 0;
}

int main(void) {
    int step = 0;
    FILE *fptr;
    fptr = fopen("simulation.txt", "w");
    //data structure initialization
    initialization();
    //get inputs
    get_instructions();
    get_registers();
    get_datamemory();
    file_print_insNode(fptr, step);
    step++;
    
    while(!Done()) {
        processRGF();
        processREB();
        processPRB();
        processAIB();
        processDAM();
        processADB();
        processSIB();
        processINB();
        
        file_print_insNode(fptr, step);
        step++;
    }
    fclose(fptr);
    return 0;
}

void processRGF() {
    if(currentREB<0) return;
    
    if(currentREB<totalREB) {
        Register[REBnode[currentREB][0]] = REBnode[currentREB][1];
        currentREB++;
    }
    
}

void processREB() {
    if(currentPRB<0 && currentAIB<0) return;
    
    if(currentPRB<totalPRB && currentPRB>=0) {
        //printf("[DEBUG] %s:%d\n",__FUNCTION__,__LINE__);
        //printf("[DEBUG] cureentPRB=%d, totalPRB=%d\n\n", currentPRB, totalPRB);
        totalREB++;
        if(currentREB < 0) currentREB = 0;
        REBnode[totalREB-1][0] = PRBnode[currentPRB].des_reg;
        REBnode[totalREB-1][1] = PRBnode[currentPRB].fst_reg * PRBnode[currentPRB].sec_reg;
        currentPRB++;
    }
    
    if(currentAIB<totalAIB && currentAIB>=0) {
        //printf("[DEBUG]%s:%d\n",__FUNCTION__,__LINE__);
        if(!strcmp(AIBnode[currentAIB].opcode, "MUL"))
            return;
        //two cases, 1:ADD, 2:SUB
        totalREB++;
        if(currentREB < 0) currentREB = 0;
        if(!strcmp(AIBnode[currentAIB].opcode, "ADD")) {
            REBnode[totalREB-1][0] = AIBnode[currentAIB].des_reg;
            REBnode[totalREB-1][1] = AIBnode[currentAIB].fst_reg + AIBnode[currentAIB].sec_reg;
        } else {
            REBnode[totalREB-1][0] = AIBnode[currentAIB].des_reg;
            REBnode[totalREB-1][1] = AIBnode[currentAIB].fst_reg - AIBnode[currentAIB].sec_reg;
        }
        currentAIB++;
        
    } else
        return;
}

void processDAM() {
    if(currentADB<0) return;
    
    if(currentADB<totalADB) {
        DataMem[ADBnode[currentADB][1]] = Register[ADBnode[currentADB][0]];
        currentADB++;
    }
}

void processADB() {
    if(currentSIB<0) return;
    
    if(currentSIB<totalSIB){
        totalADB++;
        if(currentADB < 0) currentADB = 0;
        ADBnode[totalADB-1][0] = SIBnode[currentSIB].des_reg;
        ADBnode[totalADB-1][1] = SIBnode[currentSIB].fst_reg + SIBnode[currentSIB].sec_reg;
        currentSIB++;
    }
}

void processPRB() {
    if(currentAIB<0) return;
    //must be MUL opcode and not emty for AIBnode
    if(strcmp(AIBnode[currentAIB].opcode, "MUL") == 0 && currentAIB<totalAIB) {
        totalPRB++;
        if(currentPRB < 0) currentPRB = 0;
        strcpy(PRBnode[totalPRB-1].opcode, AIBnode[currentAIB].opcode);
        PRBnode[totalPRB-1].des_reg = AIBnode[currentAIB].des_reg;
        PRBnode[totalPRB-1].fst_reg = AIBnode[currentAIB].fst_reg;
        PRBnode[totalPRB-1].sec_reg = AIBnode[currentAIB].sec_reg;
        currentAIB++;
    }
}

void processAIB() {
    if(currentINB<0) return;
    //must not be ST opcode and not emty for INBnode
    if(strcmp(INBnode[currentINB].opcode, "ST") != 0 && currentINB<totalINB) {
        totalAIB++;
        if(currentAIB < 0) currentAIB = 0;
        strcpy(AIBnode[totalAIB-1].opcode, INBnode[currentINB].opcode);
        AIBnode[totalAIB-1].des_reg = INBnode[currentINB].des_reg;
        AIBnode[totalAIB-1].fst_reg = INBnode[currentINB].fst_reg;
        AIBnode[totalAIB-1].sec_reg = INBnode[currentINB].sec_reg;
        currentINB++;
    }
}

void processSIB() {
    if(currentINB<0) return;
    //must be ST opcode and not emty for INBnode
    if(!strcmp(INBnode[currentINB].opcode, "ST") && currentINB<totalINB) {
        totalSIB++;
        if(currentSIB < 0) currentSIB = 0;
        strcpy(SIBnode[totalSIB-1].opcode, INBnode[currentINB].opcode);
        SIBnode[totalSIB-1].des_reg = INBnode[currentINB].des_reg;
        SIBnode[totalSIB-1].fst_reg = INBnode[currentINB].fst_reg;
        SIBnode[totalSIB-1].sec_reg = INBnode[currentINB].sec_reg;
        currentINB++;
    }
}

//READ for RGF
void processINB() {
    if(total_ins == 0)
        return;
    if(current_ins<total_ins){
        
        if(currentINB < 0) currentINB = 0;
        
        totalINB = totalINB + 1;
        //currentINB = currentINB + 1;
        if(!strcmp(insNode[current_ins].opcode, "ST")){
            strcpy(INBnode[totalINB-1].opcode, insNode[current_ins].opcode);
            INBnode[totalINB-1].des_reg = insNode[current_ins].des_reg;
            INBnode[totalINB-1].fst_reg = Register[insNode[current_ins].fst_reg];
            INBnode[totalINB-1].sec_reg = insNode[current_ins].sec_reg;
        } else {
            strcpy(INBnode[totalINB-1].opcode, insNode[current_ins].opcode);
            INBnode[totalINB-1].des_reg = insNode[current_ins].des_reg;
            INBnode[totalINB-1].fst_reg = Register[insNode[current_ins].fst_reg];
            INBnode[totalINB-1].sec_reg = Register[insNode[current_ins].sec_reg];
        }
        current_ins = current_ins + 1;
    }
}

void get_instructions() {
    FILE *fptr;
    if ((fptr = fopen("instructions.txt", "r")) == NULL) {
        printf("instructions.txt NOT FOUND!\n\n");
        return;
    }
    char line[24];
    char *pch;
    int i = 0;
     
    while(fgets(line, 24, fptr) != NULL ) {
        if(line[0] != '<') break; //unexpected input format
        pch = strtok(line, "<>,R");
        sscanf(pch, "%s", insNode[i].opcode);
        pch = strtok(NULL, "<>,R");
        sscanf(pch, "%d", &insNode[i].des_reg);
        pch = strtok(NULL, "<>,R");
        sscanf(pch, "%d", &insNode[i].fst_reg);
        pch = strtok(NULL, "<>,R");
        sscanf(pch, "%d", &insNode[i].sec_reg);
        i++;
    }
    total_ins = i;
    if(total_ins>0)
        current_ins = 0;
    else
        current_ins = -1;
    //printf("\n[DEUBG] Total instructions: %d\n", total_ins);
    fclose(fptr);
    return;
}

void get_registers() {
    FILE *fptr;
    if ((fptr = fopen("registers.txt", "r")) == NULL) {
        printf("registers.txt NOT FOUND!\n\n");
        return;
    }
    char line[24];
    char *pch;
    int reg_num;
    int i = 0;
     
    while(fgets(line, 24, fptr) != NULL ) {
        if(line[0] != '<') break; //unexpected input format
        pch = strtok(line, "<>,R");
        sscanf(pch, "%d", &reg_num);
        pch = strtok(NULL, "<>,R");
        sscanf(pch, "%d", &Register[reg_num]);
        i++;
    }
    fclose(fptr);
    return;
}

void get_datamemory() {
    FILE *fptr;
    if ((fptr = fopen("datamemory.txt", "r")) == NULL) {
        printf("datamemory.txt NOT FOUND!\n\n");
        return;
    }
    char line[24];
    char *pch;
    int address_num;
    int i = 0;
     
    while(fgets(line, 24, fptr) != NULL ) {
        if(line[0] != '<') break; //unexpected input format
        pch = strtok(line, "<>,R");
        sscanf(pch, "%d", &address_num);
        pch = strtok(NULL, "<>,R");
        sscanf(pch, "%d", &DataMem[address_num]);
        i++;
    }
    fclose(fptr);
    return;
}

void printINM(FILE *fptr) {
    
    int i;
    
    for(i=current_ins ; i<total_ins ; i++){
        if(i == current_ins){
            if(!strcmp(insNode[i].opcode,"ST"))
                fprintf(fptr, "<%s,R%d,R%d,%d>", insNode[i].opcode, insNode[i].des_reg, insNode[i].fst_reg, insNode[i].sec_reg);
            else
                fprintf(fptr, "<%s,R%d,R%d,R%d>", insNode[i].opcode, insNode[i].des_reg, insNode[i].fst_reg, insNode[i].sec_reg);
        } else {
            if(!strcmp(insNode[i].opcode,"ST"))
                fprintf(fptr, ",<%s,R%d,R%d,%d>", insNode[i].opcode, insNode[i].des_reg, insNode[i].fst_reg, insNode[i].sec_reg);
            else
                fprintf(fptr, ",<%s,R%d,R%d,R%d>", insNode[i].opcode, insNode[i].des_reg, insNode[i].fst_reg, insNode[i].sec_reg);
        }
    }
    fprintf(fptr, "\n");
}

void printRegister(FILE *fptr) {
    int first_print_flag = 1, i;
    for(i = 0; i<MAX ; i++){
        if(Register[i] != -1){
            if(first_print_flag){
                fprintf(fptr, "<R%d,%d>", i, Register[i]);
                first_print_flag = 0;
            } else
                fprintf(fptr, ",<R%d,%d>", i, Register[i]);
            
        }
    }
    fprintf(fptr, "\n");
}

void printDAM(FILE *fptr) {
    int first_print_flag = 1, i;
    for(i = 0; i<MAX ; i++){
        if(DataMem[i] != -1){
            if(first_print_flag){
                fprintf(fptr, "<%d,%d>", i, DataMem[i]);
                first_print_flag = 0;
            } else
                fprintf(fptr, ",<%d,%d>", i, DataMem[i]);
            
        }
    }
    fprintf(fptr, "\n");
}

void printINB(FILE *fptr) {
    int i, first_print_flag = 1;
    
    //empty for INB
    if(currentINB < 0){
        fprintf(fptr, "\n");
        return;
    }
    
    for(i=currentINB ; i<totalINB ; i++){
        if(first_print_flag){
            first_print_flag = 0;
            if(!strcmp(INBnode[i].opcode,"ST"))
                fprintf(fptr, "<%s,R%d,%d,%d>", INBnode[i].opcode, INBnode[i].des_reg, INBnode[i].fst_reg, INBnode[i].sec_reg);
            else
                fprintf(fptr, "<%s,R%d,%d,%d>", INBnode[i].opcode, INBnode[i].des_reg, INBnode[i].fst_reg, INBnode[i].sec_reg);
        } else {
            if(!strcmp(INBnode[i].opcode,"ST"))
                fprintf(fptr, ",<%s,R%d,%d,%d>", INBnode[i].opcode, INBnode[i].des_reg, INBnode[i].fst_reg, INBnode[i].sec_reg);
            else
                fprintf(fptr, ",<%s,R%d,%d,%d>", INBnode[i].opcode, INBnode[i].des_reg, INBnode[i].fst_reg, INBnode[i].sec_reg);
        }
    }
    fprintf(fptr, "\n");
}

void printSIB(FILE *fptr) {
    int i, first_print_flag = 1;
    
    //empty for SIB
    if(currentSIB < 0){
        fprintf(fptr, "\n");
        return;
    }
    
    for(i=currentSIB ; i<totalSIB ; i++){
        if(first_print_flag){
            first_print_flag = 0;
            if(!strcmp(SIBnode[i].opcode,"ST"))
                fprintf(fptr, "<%s,R%d,%d,%d>", SIBnode[i].opcode, SIBnode[i].des_reg, SIBnode[i].fst_reg, SIBnode[i].sec_reg);
            else
                fprintf(fptr, "<%s,R%d,%d,%d>", SIBnode[i].opcode, SIBnode[i].des_reg, SIBnode[i].fst_reg, SIBnode[i].sec_reg);
        } else {
            if(!strcmp(SIBnode[i].opcode,"ST"))
                fprintf(fptr, ",<%s,R%d,%d,%d>", SIBnode[i].opcode, SIBnode[i].des_reg, SIBnode[i].fst_reg, SIBnode[i].sec_reg);
            else
                fprintf(fptr, ",<%s,R%d,%d,%d>", SIBnode[i].opcode, SIBnode[i].des_reg, SIBnode[i].fst_reg, SIBnode[i].sec_reg);
        }
    }
    fprintf(fptr, "\n");
}

void printAIB(FILE *fptr) {
    int i, first_print_flag = 1;
    
    //empty for SIB
    if(currentAIB < 0){
        fprintf(fptr, "\n");
        return;
    }
    
    for(i=currentAIB ; i<totalAIB ; i++){
        if(first_print_flag){
            first_print_flag = 0;
            fprintf(fptr, "<%s,R%d,%d,%d>", AIBnode[i].opcode, AIBnode[i].des_reg, AIBnode[i].fst_reg, AIBnode[i].sec_reg);
        } else {
            fprintf(fptr, ",<%s,R%d,%d,%d>", AIBnode[i].opcode, AIBnode[i].des_reg, AIBnode[i].fst_reg, AIBnode[i].sec_reg);
        }
    }
    fprintf(fptr, "\n");
}

void printPRB(FILE *fptr) {
    int i, first_print_flag = 1;
    
    //empty for SIB
    if(currentPRB < 0){
        fprintf(fptr, "\n");
        return;
    }
    
    for(i=currentPRB ; i<totalPRB ; i++){
        if(first_print_flag){
            first_print_flag = 0;
            fprintf(fptr, "<%s,R%d,%d,%d>", PRBnode[i].opcode, PRBnode[i].des_reg, PRBnode[i].fst_reg, PRBnode[i].sec_reg);
        } else {
            fprintf(fptr, ",<%s,R%d,%d,%d>", PRBnode[i].opcode, PRBnode[i].des_reg, PRBnode[i].fst_reg, PRBnode[i].sec_reg);
        }
    }
    fprintf(fptr, "\n");
}

void printADB(FILE *fptr) {
    int i, first_print_flag = 1;
    
    if(currentADB < 0){
        fprintf(fptr, "\n");
        return;
    }
    
    for(i=currentADB ; i<totalADB ; i++){
        if(first_print_flag){
            first_print_flag = 0;
            fprintf(fptr, "<R%d,%d>", ADBnode[i][0], ADBnode[i][1]);
        } else {
            fprintf(fptr, ",<R%d,%d>", ADBnode[i][0], ADBnode[i][1]);
        }
    }
    fprintf(fptr, "\n");
}

void printREB(FILE *fptr) {
    int i, first_print_flag = 1;
    
    if(currentREB < 0){
        fprintf(fptr, "\n");
        return;
    }
    
    for(i=currentREB ; i<totalREB ; i++){
        if(first_print_flag){
            first_print_flag = 0;
            fprintf(fptr, "<R%d,%d>", REBnode[i][0], REBnode[i][1]);
        } else {
            fprintf(fptr, ",<R%d,%d>", REBnode[i][0], REBnode[i][1]);
        }
    }
    fprintf(fptr, "\n");
}
