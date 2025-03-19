# BIGINT
This C program implements BigInt arithmetic, supporting addition, subtraction, and multiplication for large integers using arrays. It includes string conversion, comparison, and dynamic memory management. The program efficiently handles large numbers beyond standard data types, ensuring precision in arithmetic operations.
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define BIGINT_SIZE 309  // Size for 1024-bit integer

typedef struct BIG_INT {
    int *num;         // Array to store the number digits in reverse order
    int signedBit;    // 1 if negative, 0 if positive
    int length;       // Length of the number
} BigInt;

// Function declarations
BigInt* addBigInts(BigInt *b1, BigInt *b2);
BigInt* subtractBigInts(BigInt *b1, BigInt *b2);
BigInt* multBigInts(BigInt *b1, BigInt *b2);
void printBigInt(BigInt *b);
BigInt stringToBigInt(char *s);
int compare(BigInt *b1, BigInt *b2);
BigInt* subtraction(BigInt *b1, BigInt *b2);

// Print a BigInt number
void printBigInt(BigInt *b) {
    int i;
    if (b->signedBit == 1) {
        printf("-");
    }
    for (i = b->length - 1; i >= 0; i--) {
        printf("%d", b->num[i]);
    }
}

// Convert a string to BigInt data type
BigInt stringToBigInt(char *s) {
    BigInt b;
    int size = strlen(s);
    if (s[0] == '-') {
        b.signedBit = 1;
    } else {
        b.signedBit = 0;
    }
    b.length = size - b.signedBit;
    b.num = (int*)malloc(sizeof(int) * b.length);
    int *i = b.num;
    size--;
    while (size >= b.signedBit) {
        *(i) = s[size] - 48;  // Convert character to integer
        i++;
        size--;
    }
    return b;
}

// Compare two BigInts
// Returns 0 if same, <0 if b2 > b1, >0 if b1 > b2
int compare(BigInt *b1, BigInt *b2) {
    int i, retval;
    if (b1->length == b2->length) {
        i = b1->length - 1;
        while (i >= 0 && b1->num[i] == b2->num[i]) {
            i--;
        }
        if (i < 0) {
            retval = 0;
        } else {
            retval = (b1->num[i]) - (b2->num[i]);
        }
    } else {
        retval = (b1->length) - (b2->length);
    }
    return retval;
}

// Subtraction helper function
BigInt* subtraction(BigInt *b1, BigInt *b2) {
    BigInt diff = (BigInt)malloc(sizeof(BigInt));
    diff->num = (int*)malloc(sizeof(int) * (b1->length));
    int borrow = 0;
    int i = 0, digit;
    while (i < b2->length) {
        digit = (b1->num[i]) - (b2->num[i]) - borrow;
        if (digit < 0) {
            borrow = 1;
            digit += 10;
        } else {
            borrow = 0;
        }
        diff->num[i] = digit;
        i++;
    }
    while (i < b1->length) {
        digit = (b1->num[i]) - borrow;
        if (digit < 0) {
            borrow = 1;
            digit += 10;
        } else {
            borrow = 0;
        }
        diff->num[i] = digit;
        i++;
    }
    diff->length = i;
    return diff;
}

// Addition of two BigInts
BigInt* addBigInts(BigInt *b1, BigInt *b2) {
    int n1 = b1->length;
    int n2 = b2->length;
    int carry;
    BigInt sum = (BigInt)malloc(sizeof(BigInt));

    if (n2 > n1)
        sum->num = (int*)malloc(sizeof(int) * (n2 + 1));
    else
        sum->num = (int*)malloc(sizeof(int) * (n1 + 1));

    // If both numbers have the same sign
    if (b1->signedBit == b2->signedBit) {
        sum->signedBit = b1->signedBit;
        carry = 0;
        int i = 0, digit;
        while (i < n1 && i < n2) {
            digit = (b1->num[i]) + (b2->num[i]) + carry;
            carry = digit / 10;
            digit = digit % 10;
            sum->num[i] = digit;
            i++;
        }
        while (i < n1) {
            digit = (b1->num[i]) + carry;
            carry = digit / 10;
            digit = digit % 10;
            sum->num[i] = digit;
            i++;
        }
        while (i < n2) {
            digit = (b2->num[i]) + carry;
            carry = digit / 10;
            digit = digit % 10;
            sum->num[i] = digit;
            i++;
        }
        while (carry != 0) {
            sum->num[i] = (carry % 10);
            carry /= 10;
            i++;
        }
        sum->length = i;
    } else {
        // Different signs, perform subtraction
        if (b1->signedBit == 1) {
            b1->signedBit = 0;
            sum = subtractBigInts(b2, b1);
            b1->signedBit = 1;
        } else {
            b2->signedBit = 0;
            sum = subtractBigInts(b1, b2);
            b2->signedBit = 1;
        }
    }
    return sum;
}

// Subtraction of two BigInts
BigInt* subtractBigInts(BigInt *b1, BigInt *b2) {
    BigInt diff = (BigInt)malloc(sizeof(BigInt));
    int n1 = b1->length;
    int n2 = b2->length;
    int borrow;
    diff->num = (int*)malloc(sizeof(int) * n1);

    if (b1->signedBit != b2->signedBit) {
        if (b2->signedBit == 1) {
            b2->signedBit = 0;
            diff = addBigInts(b1, b2);
            b2->signedBit = 1;
            diff->signedBit = 0;
        } else {
            b2->signedBit = 1;
            diff = addBigInts(b1, b2);
            b2->signedBit = 0;
            diff->signedBit = 1;
        }
    } else {
        if (compare(b1, b2) > 0) {
            diff = subtraction(b1, b2);
            diff->signedBit = b1->signedBit;
        } else {
            diff = subtraction(b2, b1);
            if (b1->signedBit == 0)
                diff->signedBit = 1;
            else
                diff->signedBit = 0;
        }
    }
    return diff;
}

// Multiplication of two BigInts
BigInt* multBigInts(BigInt *b1, BigInt *b2) {
    BigInt mult = (BigInt)malloc(sizeof(BigInt));
    mult->length = b1->length + b2->length;
    mult->num = (int*)malloc(sizeof(int) * mult->length);
    int i, j, carry, prod;

    for (i = 0; i < mult->length; i++) {
        mult->num[i] = 0;
    }

    for (i = 0; i < b1->length; i++) {
        carry = 0;
        for (j = 0; j < b2->length; j++) {
            prod = (b1->num[i] * b2->num[j]) + carry;
            carry = prod / 10;
            mult->num[i + j] += (prod % 10);
            if (mult->num[i + j] > 9) {
                mult->num[i + j + 1] += 1;
                mult->num[i + j] %= 10;
            }
        }
        while (carry != 0) {
            mult->num[i + j] += (carry % 10);
            if (mult->num[i + j] > 9) {
                mult->num[i + j + 1] += 1;
                mult->num[i + j] %= 10;
            }
            carry /= 10;
            j++;
        }
    }

    mult->signedBit = (b1->signedBit == b2->signedBit) ? 0 : 1;

    i = mult->length - 1;
    while (mult->num[i] == 0 && i >= 0) {
        i--;
    }

    mult->length = i + 1;
    if (mult->length == 0) {
        mult->signedBit = 0;
        mult->length = 1;
    }

    return mult;
}

// Main function
int main() {
    BigInt num1, num2;
    char *n1, *n2;

    n1 = (char*)malloc(sizeof(char) * BIGINT_SIZE);
    n2 = (char*)malloc(sizeof(char) * BIGINT_SIZE);

    printf("Enter number 1 : ");
    scanf("%s", n1);
    printf("Enter number 2 : ");
    scanf("%s", n2);

    num1 = stringToBigInt(n1);
    num2 = stringToBigInt(n2);

    BigInt *ans = addBigInts(&num1, &num2);
    printf("SUM OF 2 NUMBERS : ");
    printBigInt(ans);

    ans = subtractBigInts(&num1, &num2);
    printf("\nDIFFERENCE OF 2 NUMBERS : ");
    printBigInt(ans);

    ans = multBigInts(&num1, &num2);
    printf("\nPRODUCT OF 2 NUMBERS : ");
    printBigInt(ans);

    return 0;
    }
