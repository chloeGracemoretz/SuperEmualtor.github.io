# CHIP-8 Introdution


*CHIP-8 is an interpreted programming language, developed by Joseph Weisbecker. It was initially used on the COSMAC VIP and Telmac 1800 8-bit microcomputers in the mid-1970s. CHIP-8 programs are run on a CHIP-8 virtual machine. It was made to allow video games to be more easily programmed for said computers.*

*Roughly twenty years after CHIP-8 was introduced, derived interpreters appeared for some models of graphing calculators (from the late 1980s onward, these handheld devices in many ways have more computing power than most mid-1970s microcomputers for hobbyists).*

## Virtual machine description
### Memory
*CHIP-8 was most commonly implemented on 4K systems, such as the Cosmac VIP and the Telmac 1800. These machines had 4096 (0x1000) memory locations, all of which are 8 bits (a byte) which is where the term CHIP-8 originated. However, the CHIP-8 interpreter itself occupies the first 512 bytes of the memory space on these machines. For this reason, most programs written for the original system begin at memory location 512 (0x200) and do not access any of the memory below the location 512 (0x200). The uppermost 256 bytes (0xF00-0xFFF) are reserved for display refresh, and the 96 bytes below that (0xEA0-0xEFF) were reserved for call stack, internal use, and other variables.*

### Registers
*CHIP-8 has 16 8-bit data registers named from V0 to VF. The VF register doubles as a flag for some instructions, thus it should be avoided. In addition operation VF is for carry flag. While in subtraction, it is the "no borrow" flag. In the draw instruction the VF is set upon pixel collision.The address register, which is named I, is 16 bits wide and is used with several opcodes that involve memory operations.*

### The stack
*The stack is only used to store return addresses when subroutines are called. The original 1802 version allocated 48 bytes for up to 24 levels of nesting; modern implementations normally have at least 16 levels.*

### Opcode table
*CHIP-8 has 35 opcodes, which are all two bytes long and stored big-endian. The opcodes are listed below, in hexadecimal and with the following symbols:*

* NNN: address
* NN: 8-bit constant
* N: 4-bit constant
* X and Y: 4-bit register identifier
* PC : Program Counter
* I : 16bit register (For memory address) (Similar to void pointer)

## 部分c代码实现：
```
#ifndef CHIP8_CPU_H
#define CHIP8_CPU_H
#define r_size 8
#define v_size 16
#define stack_size 16
#define OneByte 256
#define pixels_size 8192

#endif //CHIP8_CPU_H
typedef struct chip8CentralProcessingUnit {
    int x, y, n, nn, nnn, sp, PC, opCode, I, key, DT,ST, soundTimer,pixelHeight,pixelWidth;
    int R[r_size], V[v_size], stack[stack_size], pixels[pixels_size];
    bool bufferFlag, pressed,mode;

    int (*getRandom)();

    void (*decodeInstruction)(struct chip8CentralProcessingUnit *pthis, int instruction);

    void (*proceed)(struct chip8CentralProcessingUnit *pthis,int *memory);

    void (*executeInstruction)(struct chip8CentralProcessingUnit *pthis,int *memory);
} __CPU;

int getRandom();

void initCPU(__CPU *);

void decodeInstruction(__CPU *, int);

void proceed(__CPU*,int*);

void executeInstruction(__CPU *,int *);

//int NoIns = 0;
int getRandom() {
    return rand() % OneByte;
}

void initCPU(__CPU *pthis) {
    pthis->PC = 0x200;
    pthis->getRandom = &getRandom;
    pthis->decodeInstruction = &decodeInstruction;
    pthis->executeInstruction = &executeInstruction;
    pthis->proceed = &proceed;
    pthis->DT = 0;
    pthis->I = 0;
    pthis->key = -1;
    pthis->pixelWidth = 128;
    pthis->pixelHeight = 64;
    //pthis->bufferFlag = true; //for test
    memset(pthis->R, 0, sizeof(pthis->R));

    memset(pthis->V, 0, sizeof(pthis->V));

    memset(pthis->stack, 0, sizeof(pthis->stack));
    memset(pthis->pixels, 0, sizeof(pthis->pixels));
}

//cpu execute one piece of instructions
void proceed(__CPU *pthis,int *memory) {
    int instruction = ((*(memory+ pthis->PC ) & 0xff) << 8) | (*(memory+pthis->PC + 0x1) & 0xff);
    pthis->decodeInstruction(pthis, instruction);
    pthis->PC += 2;
    pthis->executeInstruction(pthis,memory);
    if(pthis->DT>0)
        pthis->DT--;

}

//decode the instruction and get the paramters
void decodeInstruction(__CPU *pthis, int instruction) {
    pthis->x = (instruction >> 8) & 0x000f;
    pthis->y = (instruction >> 4) & 0x000f;
    pthis->n = (instruction) & 0x000f;
    pthis->nn = (instruction) & 0x00ff;
    pthis->nnn = (instruction) & 0x0fff;
    pthis->opCode = (instruction >> 12) & 0x000f;
    //printf("No:%d instruction: %x x:%x y:%x n:%x nn:%x nnn:%x opCode:%x\n",NoIns,instruction, pthis->x, pthis->y, pthis->n, pthis->nn, pthis->nnn, pthis->opCode);
    //NoIns++;
}


//execute the commands based on the paramters
void executeInstruction(__CPU *pthis,int *memory) {
    if (pthis->opCode == 0x00 && pthis->nnn == 0xe0) {
        for (int p = 0; p < pixels_size; ++p) pthis->pixels[p] = 0;
        pthis->bufferFlag = true;
        return;
    }
    if (pthis->opCode == 0x00 && pthis->nnn == 0xee) {
        pthis->PC = pthis->stack[--pthis->sp % 16];
        return;
    }
    if (pthis->opCode == 0x01) {
        pthis->PC = pthis->nnn;
        return;
    }
    if (pthis->opCode == 0x02) {
        pthis->stack[pthis->sp++ % 16] = pthis->PC;
        pthis->PC = pthis->nnn;
        return;
    }
    if (pthis->opCode == 0x03) {
        if (pthis->V[pthis->x] == pthis->nn) pthis->PC += 2;
        return;
    }
    if (pthis->opCode == 0x04) {
        if (pthis->V[pthis->x] != pthis->nn) pthis->PC += 2;
        return;
    }
    if (pthis->opCode == 0x05 && pthis->n == 0x00) {
        if (pthis->V[pthis->x] == pthis->V[pthis->y]) pthis->PC += 2;
        return;
    }
    if (pthis->opCode == 0x06) {
        pthis->V[pthis->x] = pthis->nn & 0xff;
        return;
    }
    if (pthis->opCode == 0x07) {
        pthis->V[pthis->x] = (pthis->V[pthis->x] + pthis->nn) & 0xff;
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x00) {
        pthis->V[pthis->x] = pthis->V[pthis->y];
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x01) {
        pthis->V[pthis->x] |= pthis->V[pthis->y];
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x02) {
        pthis->V[pthis->x] &= pthis->V[pthis->y];
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x03) {
        pthis->V[pthis->x] ^= pthis->V[pthis->y];
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x04) {
        pthis->V[pthis->x] += pthis->V[pthis->y];
        pthis->V[0xf] = (pthis->V[pthis->x] >> 8) & 1;
        pthis->V[pthis->x] &= 0xff;
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x05) {
        pthis->V[0xf] = pthis->V[pthis->x] >= pthis->V[pthis->y] ? 1 : 0;
        pthis->V[pthis->x] -= pthis->V[pthis->y];
        pthis->V[pthis->x] &= 0xff;
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x06) {
        pthis->V[0xf] = pthis->V[pthis->x] & 0x1;
        pthis->V[pthis->x] = (pthis->V[pthis->x] >> 1) & 0xff;
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x07) {
        pthis->V[0xf] = pthis->V[pthis->x] >= pthis->V[pthis->y] ? 1 : 0;
        pthis->V[pthis->x] = (pthis->V[pthis->y] - pthis->V[pthis->x]) & 0xff;
        return;
    }
    if (pthis->opCode == 0x08 && pthis->n == 0x0e) {
        pthis->V[0xf] = (pthis->V[pthis->x] >> 7) & 1;
        pthis->V[pthis->x] <<= 1;
        pthis->V[pthis->x] &= 0xff;
        return;
    }
    if (pthis->opCode == 0x09 && pthis->n == 0x00) {
        if (pthis->V[pthis->x] != pthis->V[pthis->y]) pthis->PC += 2;
        return;
    }
    if (pthis->opCode == 0x0a) {
        pthis->I = pthis->nnn;
        return;
    }
    if (pthis->opCode == 0x0b) {
        pthis->PC = pthis->nnn + pthis->V[0];
        return;
    }
    if (pthis->opCode == 0x0c) {
        pthis->V[pthis->x] = pthis->getRandom() & pthis->nn;
        return;
    }
    //store the image into pixels
    if (pthis->opCode == 0x0d && pthis->n != 0) {
        int gg;
        for (int j = 0; j < pthis->n; ++j)
            for (int i = 0; i < 8; ++i) {
                if ((*(memory + pthis->I + j) & (0x80 >> i)) != 0) {
                    gg = (pthis->V[pthis->x] + i + ((pthis->V[pthis->y] + j) * pthis->pixelWidth)) % (pthis->pixelWidth * pthis->pixelHeight);
                    pthis->V[0xf] = pthis->pixels[gg] & 0x1;
                    pthis->pixels[gg] ^= 0x1;
                }
            }
        pthis->bufferFlag = true;
        return;
    }
    if (pthis->opCode == 0x0e && pthis->nn == 0x9e) {
        if (pthis->key == pthis->V[pthis->x]) pthis->PC += 2;
        return;
    }
    if (pthis->opCode == 0x0e && pthis->nn == 0xa1) {
        if (pthis->key != pthis->V[pthis->x]) pthis->PC += 2;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x07) {
        pthis->V[pthis->x] = pthis->DT & 0x00ff;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x0a) {
        if (pthis->pressed) {
            pthis->V[pthis->x] = pthis->key & 0xff;
            pthis->pressed = true;
        } else { pthis->PC -= 2; }
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x15) {
        pthis->DT = pthis->V[pthis->x] & 0x00ff;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x18) {
        pthis->soundTimer = pthis->V[pthis->x] & 0x00ff;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x1e) {
        pthis->I += pthis->V[pthis->x];
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x29) {
        pthis->I = pthis->V[pthis->x] * 5;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x33) {
        *(memory + (pthis->I & 0xfff)) = ((pthis->V[pthis->x] / 100) % 10) & 0xf;
        *(memory + ((pthis->I + 1) & 0xfff)) = ((pthis->V[pthis->x] / 10) % 10) & 0xf;
        *(memory + ((pthis->I + 2) & 0xfff)) = ((pthis->V[pthis->x]) % 10) & 0xf;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x55) {
        for (int p = 0; p <= pthis->x; ++p)
            *(memory + pthis->I + p) = pthis->V[p] & 0xff;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn == 0x65) {
        for (int p = 0; p <= pthis->x; ++p)
            pthis->V[p] = *(memory + pthis->I + p) & 0xff;
        return;
    }
    if (pthis->opCode == 0x00 && pthis->x == 0x00 && pthis->y == 0x0b ) {
        for (int mm = 0; mm < 8192 - pthis->pixelWidth * pthis->n; ++mm) {
            pthis->pixels[mm] = pthis->pixels[mm + 128 * pthis->n];
        }
        for (int pp = 8191 - pthis->pixelWidth * pthis->n; pp < 8192; ++pp) pthis->pixels[pp] = 0;
        pthis->bufferFlag = true;
        return;
    }
    if (pthis->opCode == 0x00 &&pthis-> x == 0x00 && pthis->y == 0x0c ) {
        for (int mm = 8191; mm >= pthis->pixelWidth * pthis->n; --mm) {
            pthis->pixels[mm] = pthis->pixels[mm - pthis->pixelWidth *pthis-> n];
        }
        for (int pp = 0; pp < pthis->pixelWidth *pthis-> n; ++pp) pthis-> pixels[pp] = 0;
        pthis-> bufferFlag = true;
    }
    if (pthis->opCode == 0x00 && pthis->nnn == 0xfb ) {
        int i, j, k;
        for (i = 0; i < pthis->pixelHeight; ++i) {
            for (j = pthis->pixelWidth - 1; j >= 4; --j) {
                pthis->pixels[j + i * pthis->pixelWidth] = pthis->pixels[j - 4 + i * pthis->pixelWidth];
            }
            for (k = 3; k >= 0; --k) pthis->pixels[i * pthis->pixelWidth + k] = 0;
        }
        pthis->bufferFlag = true;
    }
    if (pthis->opCode == 0x00 && pthis->nnn == 0xfc ) {
        int i, j, k;
        for (i = 0; i < pthis->pixelHeight; ++i) {
            for (j = 0; j <pthis->pixelWidth - 4; ++j) {
                pthis-> pixels[j + i * pthis->pixelWidth] = pthis->pixels[j + 4 + i * pthis->pixelWidth];
            }
            for (k = pthis->pixelWidth - 4; k < pthis->pixelWidth; ++k) pthis->pixels[i * pthis->pixelWidth + k] = 0;
        }
        pthis->bufferFlag = true;
        return;
    }
    if (pthis->opCode == 0x00 && pthis->nnn == 0xfd ) { exit(0); }
    if (pthis->opCode == 0x00 && pthis->nnn == 0xfe ) { pthis->pixelWidth = 64;pthis-> pixelHeight = 32; pthis->mode = false; return; }    // false - 64x32  mode
    if (pthis->opCode == 0x00 && pthis->nnn == 0xff ) { pthis->pixelWidth = 128; pthis->pixelHeight = 64; pthis->mode = true; return; }    // true  - 124x64 mode
    if (pthis->opCode == 0x05 && pthis->n == 0x01 ) { if (pthis->V[pthis->x] > pthis->V[pthis->y]) pthis->PC += 2; return; }
    if (pthis->opCode == 0x05 && pthis->n == 0x03 ) { if (pthis->V[pthis->x] < pthis->V[pthis->y]) pthis->PC += 2; return; }
    if (pthis->opCode == 0x09 && pthis->n == 0x01 ) { int tmp = pthis->V[pthis->x] * pthis->V[pthis->y]; pthis->V[0xf] = (tmp >> 8) & 0xff; pthis->V[pthis->x] = tmp & 0xff; return; }
    if (pthis->opCode == 0x09 && pthis->n == 0x02 ) { int tmp = pthis->V[pthis->x] / pthis->V[pthis->y]; pthis->V[0xf] = 0; pthis->V[pthis->x] = tmp & 0xff; return; }
    if (pthis->opCode == 0x09 && pthis->n == 0x03 ) {
        *(memory + (pthis-> I & 0xfff)) = ((pthis->V[pthis->x] / 10000) % 10) & 0xf;
        *(memory +((pthis->I + 1) & 0xfff)) = ((pthis->V[pthis->x] / 1000 ) % 10) & 0xf;
        *(memory + ((pthis->I + 2) & 0xfff)) = ((pthis->V[pthis->x] / 100  ) % 10) & 0xf;
        *(memory+ ((pthis->I + 3) & 0xfff)) = ((pthis->V[pthis->x] / 10   ) % 10) & 0xf;
        *(memory + ((pthis->I + 4) & 0xfff)) = ((pthis->V[pthis->x]        ) % 10) & 0xf;
        return;
    }
    if (pthis->opCode == 0x0d && pthis->n == 0   ) {
        int dword, gg;
        for (int j = 0; j < 16; j += 1) {
            dword =  (*(memory + pthis->I + j * 2) << 8) | (*(memory + pthis->I + j * 2 + 1) & 0xff);
            for (int i = 0; i < 16; ++i) {
                if (( dword & (0x8000 >> i)) != 0) {
                    gg = (pthis->V[pthis->x] + i + ((pthis->V[pthis->y] + j) * pthis->pixelWidth)) % (pthis->pixelWidth * pthis->pixelHeight);
                    pthis->V[0xf] =pthis-> pixels[gg] & 0x1;
                    pthis-> pixels[gg] ^= 0x1;
                }
            }
        }
        pthis->bufferFlag = true;
        return;
    }
    if (pthis->opCode == 0x0f && pthis->nn  == 0x30 ) { pthis->I = pthis->V[pthis->x] * 10 + 0x50; return; }
    if (pthis->opCode == 0x0f && pthis->nn  == 0x75 ) { for (int p = 0; p <= pthis->x; ++p) pthis->R[p] = pthis->V[p]; return; }
    if (pthis->opCode == 0x0f && pthis->nn  == 0x85 ) { for (int p = 0; p <= pthis->x; ++p) pthis->V[p] = pthis->R[p]; return; }
    if (pthis->opCode == 0x0f && pthis->nn  == 0x94 ) { pthis->I = pthis->V[pthis->x] * 3 + 0x100; }
}
```
