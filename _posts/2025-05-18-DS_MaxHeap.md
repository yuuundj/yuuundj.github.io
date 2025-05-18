---
layout: single
title:  "실시간 차트"
math: true
---

# Max Heap의 활용
## 빈도수 상위 5개의 단어를 출력
단어와 발생 시각에 대한 정보가 있는 txt 파일을 읽고 특정시간까지의 빈도수 상위 5개의 단어를 매 시간 출력한다.

단, 같은 빈도수의 경우 알파벳 오름차순으로 출력된다.

## Max Heap의 사용
빈도수가 가장 많은 단어를 출력하기 위해 부모 노드가 자식 노드보다 항상 큰 Max Heap을 활용한다.

단어의 빈도수가 같은 경우 알파벳 오름차순으로 출력되어야 하므로 Heap은 빈도수 - 알파벳 오름차순의 우선도로 정렬되어야 한다.

Max Heap 배열 하나만 사용하는 경우, Heap은 부모 노드와 자식 노드 간 크기 차이만 있고 어느 자식 노드가 더 큰지는 알 수 없기 때문에 Heap 안에서 이미 있는 단어를 찾을 때 선형 탐색을 해야 한다.

## Max Heap과 이진 트리의 동시 사용
이진 트리는 부모 노드 크기를 기준으로 각 자식 노드가 한 쪽은 부모 노드보다 작고 다른 한 쪽은 부모 노드보다 크기 때문에 탐색의 범위를 반으로 좁힐 수 있는 특성이 있으므로 단어를 이진 트리에 알파벳 오름차순으로 정렬하여 원하는 단어를 더 빠르게 찾을 수 있다.

이진 탐색으로 단어를 찾고 그 단어의 빈도수에 변동이 생겼으니 Heap이 재정렬되어야 하는데, Heap을 포인터 배열로 구성해도 현재 그 단어의 Heap 안에서의 위치를 알 수 없기 때문에 Heap 안에서의 위치를 나타내는 변수도 같이 포함해야 한다.

## C 코드의 구성
### 활용되는 자료형
단어와 그 단어의 빈도수를 담는 구조체가 필요하며, Heap에서의 위치를 알 수 있는 인덱스, 트리로 활용하기 위한 left, right 포인터도 포함해야 한다.

Heap은 구조체 포인터 배열로 구성되며, 단어를 Heap에서 뽑아 출력하는 과정이 매 시간 일어나므로 뽑았던 단어들을 다시 Heap에 넣어주기 위한 임시 저장 배열을 구조체 포인터 배열로 구성한다.
```c
// 단어, 빈도수, 힙에서의 인덱스가 있는 구조체, 이진 트리의 노드로도 활용
typedef struct data
{
    char w[100];
    int bins;
    int idx;
    struct data *left;
    struct data *right;
} DATA;

// 단어를 사전순으로 저장할 트리의 루트 노드
DATA *root = NULL;

// 노드의 주소를 가지고 있는 힙
#define SZ 3000
DATA *heap[SZ];
int hidx = 0;

// 단어를 출력할 때 힙에서 뽑고 다시 넣어주기 위해 임시로 사용할 배열
DATA *temp[SZ];
int tidx = 0;
```

### Max Heap의 정렬 - Up Heap
현재 단어를 담고 있는 구조체의 주소를 받아서 사용한다.

현재 단어의 Heap안에서의 위치부터 빈도수와 알파벳 오름차순을 기준으로 정렬을 시작한다.

Heap안에서 현재 단어의 위치를 알기 위해 구조체 내에 Heap에서의 인덱스 값을 참조하며, 정렬하는 과정에 있어서 해당 Heap인덱스의 주소만 교환하게 되므로 교환되는 각 구조체의 Heap 인덱스 값도 갱신해주어야 한다.
```c
// upHeap
void upHeap(DATA *_c)
{
    int i = _c->idx;
    while (i > 1)
    {
        // 현재 노드보다 부모 노드의 빈도수가 더 작은 경우 교환
        if (heap[i]->bins > heap[i / 2]->bins)
        {
            heap[i]->idx = i / 2;
            heap[i / 2]->idx = i;
            DATA *tmp = heap[i];
            heap[i] = heap[i / 2];
            heap[i / 2] = tmp;
            i = i / 2;
        }
        // 현재 노드와 부모 노드의 빈도수가 같은 경우 사전 오름차순 교환
        else if (heap[i]->bins == heap[i / 2]->bins 
        && strcmp(heap[i]->w, heap[i / 2]->w) < 0)
        {
            heap[i]->idx = i / 2;
            heap[i / 2]->idx = i;
            DATA *tmp = heap[i];
            heap[i] = heap[i / 2];
            heap[i / 2] = tmp;
            i = i / 2;
        }
        else
        {
            break;
        }
    }
    return;
}
```

### Max Heap의 정렬 - Down Heap
Heap에서 가장 마지막 데이터를 1번 인덱스로 가져와서 단어의 빈도수와 알파벳 오름차순을 기준으로 정렬을 시작한다.

이때 Heap은 포인터 배열이므로 가장 마지막 데이터가 있던 곳은 참조하지 않도록 NULL로 처리하며, 인덱스를 가리키는 변수가 Heap에서 데이터가 있는 범위를 넘어서 가리키는 경우 그 전 값을 가리킬 수 있도록 처리한다.

자식 노드 중 빈도수 - 알파벳 오름차순의 우선도를 따져 더 큰 쪽을 선택하여 부모 노드와 비교하며, 두 노드의 교환 과정에서 각 구조체의 Heap인덱스도 역시 갱신해주어야 한다.
```c
// downHeap
void downHeap()
{
    heap[1] = heap[hidx];
    heap[hidx] = NULL;
    hidx--;
    int i = 1;
    while ((i * 2) <= hidx)
    {
        int large;
        // 힙 인덱스를 넘어서는 값 참조 방지
        if (i * 2 + 1 > hidx)
        {
            large = i * 2;
        }
        else
        {
            // 자식 노드 중 빈도수가 더 큰 값 선택
            large = (heap[i * 2]->bins > heap[i * 2 + 1]->bins) ? (i * 2) : (i * 2 + 1);
            // 자식 노드의 빈도수가 같은 경우 사전순이 더 빠른쪽 선택
            if (heap[i * 2]->bins == heap[i * 2 + 1]->bins)
            {
                large = (strcmp(heap[i * 2]->w, heap[i * 2 + 1]->w) > 0) ? 
                (i * 2 + 1) : (i * 2);
            }
        }
        // 현재 노드보다 자식 노드의 빈도수가 더 큰 경우 교환
        if (heap[i]->bins < heap[large]->bins)
        {
            heap[i]->idx = large;
            heap[large]->idx = i;
            DATA *tmp = heap[i];
            heap[i] = heap[large];
            heap[large] = tmp;
            i = large;
        }
        // 현재 노드와 자식 노드 빈도수가 같은 경우 사전 오름차순 교환
        else if (heap[i]->bins == heap[large]->bins)
        {
            if (strcmp(heap[i]->w, heap[large]->w) > 0)
            {
                heap[i]->idx = large;
                heap[large]->idx = i;
                DATA *tmp = heap[i];
                heap[i] = heap[large];
                heap[large] = tmp;
                i = large;
            }
            else
            {
                break;
            }
        }
        else
        {
            break;
        }
    }
    return;
}
```

### 단어의 추가
현재 단어를 트리와 Heap에 추가하는 과정이다.

알파벳 오름차순으로 정렬된 이진 트리를 구성하기 위한 공간을 할당받으며, 이진 탐색을 통해 현재 단어의 존재 유무를 판단한다.

현재 단어가 처음 등장한 단어라면 이진 트리 구조에 맞춰서 데이터가 삽입되고, 현재 단어가 이미 있는 단어라면 빈도수를 올린다.

이후 upHeap함수에 현재 단어의 정보가 있는 구조체를 넘겨 현재 단어의 Heap에서의 위치에서부터 정렬을 시작한다.
```c
// 단어를 트리와 힙에 추가
void addData(char _w[])
{
    hidx++;
    DATA *new_data = (DATA *)malloc(sizeof(DATA));
    new_data->bins = 1;
    new_data->idx = hidx;
    strcpy(new_data->w, _w);
    new_data->left = new_data->right = NULL;

    // 맨 첫 단어일 경우
    if (root == NULL)
    {
        root = new_data;
        heap[hidx] = new_data;
        return;
    }

    // 맨 첫 단어가 아닌 경우
    DATA *cur = root;
    while (cur != NULL)
    {
        // 추가할 단어가 사전순으로 더 빠른 경우
        if (strcmp(_w, cur->w) < 0)
        {
            if (cur->left == NULL)
            {
                cur->left = new_data;
                heap[hidx] = new_data;
                cur = cur->left;
                break;
            }
            cur = cur->left;
        }
        // 추가할 단어가 사전순으로 더 느린 경우
        else if (strcmp(_w, cur->w) > 0)
        {
            if (cur->right == NULL)
            {
                cur->right = new_data;
                heap[hidx] = new_data;
                cur = cur->right;
                break;
            }
            cur = cur->right;
        }
        // 추가할 단어가 이미 있는 단어일 경우
        else
        {
            hidx--;
            cur->bins++;
            free(new_data);
            break;
        }
    }
    upHeap(cur);
    return;
}
```

### 임시 배열의 활용
Heap에서 뽑았던 단어를 임시로 저장하는 배열이다.

Heap에서 가장 우선순위가 높은 데이터를 뽑아서 사용했다면 Down Heap 과정을 거쳐서 다음 우선순위가 높은 데이터가 Heap의 1번 인덱스로 오게 된다.

이 과정에서 원래 1번 인덱스였던 데이터는 사용한 후 갈 곳을 잃게 되는데, 매 시간마다 데이터를 출력해야 하므로 뽑았던 데이터를 다시 Heap에 넣어주는 과정이 필요하다.

따라서 뽑았던 데이터를 임시 배열에 저장해주어 다시 Heap에 추가할 수 있도록 하며, 이때 추가된 데이터는 Up Heap과정을 거쳐 자신의 우선도에 맞는 위치에 찾아갈 수 있도록 한다.
```c
// 임시 배열에 단어 저장
void addTemp()
{
    temp[tidx] = heap[1];
    tidx++;
    return;
}

// 임시 배열에 저장된 단어를 힙에 다시 추가
void deleteTemp()
{
    while (tidx != 0)
    {
        tidx--;
        hidx++;
        heap[hidx] = temp[tidx];
        temp[tidx] = NULL;
        heap[hidx]->idx = hidx;
        upHeap(heap[hidx]);
    }
    return;
}
```

### 상위 빈도수 5개 단어 출력
상위 빈도수 5개 단어를 출력하는데, 이때 5번째로 출력되는 단어의 빈도수가 여러개일 경우 해당 빈도수의 데이터를 모두 출력한다.

Heap에서 부모 노드인 1번 인덱스의 빈도수와 자식 노드인 2, 3번 인덱스의 빈도수를 비교하여 부모와 두 자식 노드의 빈도수가 다른 경우에 순위를 구분한다.

이때 총 몇 개의 단어가 출력되었는지 판단하는 변수를 통해 5개 이상 출력되었다면 더 이상의 다른 빈도수는 필요없으므로 출력을 종료한다.

우선도가 가장 높은 단어를 출력한 후 Down Heap 과정을 통해 데이터를 재정렬하여 다음으로 우선도가 높은 단어가 Heap의 가장 앞으로 올 수 있게 하며, 이때 출력했던 단어는 임시 배열에 저장해뒀다가 모든 출력이 끝난 후에 다시 Heap에 넣어주는 과정을 거친다.
```c
// top 5에서의 같은 빈도 수 모두 출력
void showHot()
{
    int i = 0;
    int printed = 0;
    while (i < 5 && hidx != 0)
    {
        // 최상위 단어 출력
        printf("%d. %s : %d회\n", i + 1, heap[1]->w, heap[1]->bins);
        printed++;

        // 최상위 단어와 그 아래 단어의 빈도수가 다른 경우 순위 구분
        if (heap[2] != NULL && heap[3] != NULL 
        && heap[1]->bins != heap[2]->bins && heap[1]->bins != heap[3]->bins)
        {
            printf("\n");
            i++;
            // 출력개수가 5개 이상인 경우 종료
            if (printed >= 5)
            {
                break;
            }
        }
        // 출력한 단어를 임시 배열에 저장
        addTemp();

        // 단어 빈도수 정렬
        downHeap();
    }
    // 임시 저장했던 단어들을 다시 힙에 저장
    deleteTemp();

    printf("\n");
    return;
}
```

### 동적 할당 해제
트리를 구성하기 위해 받았던 공간을 반납한다.
```c
// 동적 할당 해제
void freeTree(DATA *_r)
{
    if (_r == NULL)
    {
        return;
    }
    freeTree(_r->left);
    freeTree(_r->right);
    free(_r);
}
```

### main 함수
txt 파일을 열고 While 루프 안에서 줄마다 단어와 발생시각을 읽은 후 단어는 Heap과 이진 트리에 활용한다.

매 시간마다 빈도수 상위 5개 단어를 출력하며, 입력 받은 시간에 도달할 경우 루프를 종료한다.

루프 종료 이후에는 읽었던 파일을 닫고, 트리의 동적 할당을 해제한다.
```c
 int main()
{

    // 기준 시간
    int t = 0;
    scanf("%d", &t);
    // 입력 받은 시간이 1보다 작은 경우 종료
    if (t < 1)
    {
        printf("잘못된 시간입니다.\n");
        return 0;
    }

    // word_data.txt 파일 열기
    FILE *f = fopen("word_data.txt", "rt"); // read text
    if (f == NULL)
    {
        printf("unable to open file");
        return 0;
    }

    // 문자열과 숫자 분리
    char word[100];
    int time;

    while (1)
    {
        // 단어와 시간 분리
        int element = fscanf(f, "%s %d", word, &time);

        // 읽을 게 없으면 종료
        if (element <= 0)
        {
            break;
        }

        // 단어를 트리와 힙에 추가
        // 이미 있는 단어인 경우 빈도 수 올리고 정렬, 없는 단어인 경우 추가 후 정렬
        addData(word);

        // 빈도수 상위 5개 출력
        printf("t = %d일때 순위\n", time);
        showHot();

        // 시간 도달 시 종료
        if (time == t)
        {
            break;
        }
    }

    // 파일 닫기
    fclose(f);

    // 동적 할당 해제
    freeTree(root);
    return 0;
}
```

## 전체 코드
```c
// 파일 "word_data.txt"에 있는 단어들 중 최대 빈도 5개 뽑기
// 이진트리로 문자열 사전순 정렬 후 빈도수는 힙 활용
// 힙 안에서 빈도수 같을 때 사전순 정렬 유지

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 단어, 빈도수, 힙에서의 인덱스가 있는 구조체, 이진 트리의 노드로도 활용
typedef struct data
{
    char w[100];
    int bins;
    int idx;
    struct data *left;
    struct data *right;
} DATA;

// 단어를 사전순으로 저장할 트리의 루트 노드
DATA *root = NULL;

// 노드의 주소를 가지고 있는 힙
#define SZ 3000
DATA *heap[SZ];
int hidx = 0;

// 단어를 출력할 때 힙에서 뽑고 다시 넣어주기 위해 임시로 사용할 배열
DATA *temp[SZ];
int tidx = 0;

// upHeap
void upHeap(DATA *_c)
{
    int i = _c->idx;
    while (i > 1)
    {
        // 현재 노드보다 부모 노드의 빈도수가 더 작은 경우 교환
        if (heap[i]->bins > heap[i / 2]->bins)
        {
            heap[i]->idx = i / 2;
            heap[i / 2]->idx = i;
            DATA *tmp = heap[i];
            heap[i] = heap[i / 2];
            heap[i / 2] = tmp;
            i = i / 2;
        }
        // 현재 노드와 부모 노드의 빈도수가 같은 경우 사전 오름차순 교환
        else if (heap[i]->bins == heap[i / 2]->bins && strcmp(heap[i]->w, heap[i / 2]->w) < 0)
        {
            heap[i]->idx = i / 2;
            heap[i / 2]->idx = i;
            DATA *tmp = heap[i];
            heap[i] = heap[i / 2];
            heap[i / 2] = tmp;
            i = i / 2;
        }
        else
        {
            break;
        }
    }
    return;
}

// downHeap
void downHeap()
{
    heap[1] = heap[hidx];
    heap[hidx] = NULL;
    hidx--;
    int i = 1;
    while ((i * 2) <= hidx)
    {
        int large;
        // 힙 인덱스를 넘어서는 값 참조 방지
        if (i * 2 + 1 > hidx)
        {
            large = i * 2;
        }
        else
        {
            // 자식 노드 중 빈도수가 더 큰 값 선택
            large = (heap[i * 2]->bins > heap[i * 2 + 1]->bins) ? (i * 2) : (i * 2 + 1);
            // 자식 노드의 빈도수가 같은 경우 사전순이 더 빠른쪽 선택
            if (heap[i * 2]->bins == heap[i * 2 + 1]->bins)
            {
                large = (strcmp(heap[i * 2]->w, heap[i * 2 + 1]->w) > 0) ? (i * 2 + 1) : (i * 2);
            }
        }
        // 현재 노드보다 자식 노드의 빈도수가 더 큰 경우 교환
        if (heap[i]->bins < heap[large]->bins)
        {
            heap[i]->idx = large;
            heap[large]->idx = i;
            DATA *tmp = heap[i];
            heap[i] = heap[large];
            heap[large] = tmp;
            i = large;
        }
        // 현재 노드와 자식 노드 빈도수가 같은 경우 사전 오름차순 교환
        else if (heap[i]->bins == heap[large]->bins)
        {
            if (strcmp(heap[i]->w, heap[large]->w) > 0)
            {
                heap[i]->idx = large;
                heap[large]->idx = i;
                DATA *tmp = heap[i];
                heap[i] = heap[large];
                heap[large] = tmp;
                i = large;
            }
            else
            {
                break;
            }
        }
        else
        {
            break;
        }
    }
    return;
}

// 단어를 트리와 힙에 추가
void addData(char _w[])
{
    hidx++;
    DATA *new_data = (DATA *)malloc(sizeof(DATA));
    new_data->bins = 1;
    new_data->idx = hidx;
    strcpy(new_data->w, _w);
    new_data->left = new_data->right = NULL;

    // 맨 첫 단어일 경우
    if (root == NULL)
    {
        root = new_data;
        heap[hidx] = new_data;
        return;
    }

    // 맨 첫 단어가 아닌 경우
    DATA *cur = root;
    while (cur != NULL)
    {
        // 추가할 단어가 사전순으로 더 빠른 경우
        if (strcmp(_w, cur->w) < 0)
        {
            if (cur->left == NULL)
            {
                cur->left = new_data;
                heap[hidx] = new_data;
                cur = cur->left;
                break;
            }
            cur = cur->left;
        }
        // 추가할 단어가 사전순으로 더 느린 경우
        else if (strcmp(_w, cur->w) > 0)
        {
            if (cur->right == NULL)
            {
                cur->right = new_data;
                heap[hidx] = new_data;
                cur = cur->right;
                break;
            }
            cur = cur->right;
        }
        // 추가할 단어가 이미 있는 단어일 경우
        else
        {
            hidx--;
            cur->bins++;
            free(new_data);
            break;
        }
    }
    upHeap(cur);
    return;
}

// 임시 배열에 단어 저장
void addTemp()
{
    temp[tidx] = heap[1];
    tidx++;
    return;
}

// 임시 배열에 저장된 단어를 힙에 다시 추가
void deleteTemp()
{
    while (tidx != 0)
    {
        tidx--;
        hidx++;
        heap[hidx] = temp[tidx];
        temp[tidx] = NULL;
        heap[hidx]->idx = hidx;
        upHeap(heap[hidx]);
    }
    return;
}

// top 5에서의 같은 빈도 수 모두 출력
void showHot()
{
    int i = 0;
    int printed = 0;
    while (i < 5 && hidx != 0)
    {
        // 최상위 단어 출력
        printf("%d. %s : %d회\n", i + 1, heap[1]->w, heap[1]->bins);
        printed++;

        // 최상위 단어와 그 아래 단어의 빈도수가 다른 경우 순위 구분
        if (heap[2] != NULL && heap[3] != NULL && heap[1]->bins != heap[2]->bins && heap[1]->bins != heap[3]->bins)
        {
            printf("\n");
            i++;
            // 출력개수가 5개 이상인 경우 종료
            if (printed >= 5)
            {
                break;
            }
        }

        // 출력한 단어를 임시 배열에 저장
        addTemp();

        // 단어 빈도수 정렬
        downHeap();
    }
    // 임시 저장했던 단어들을 다시 힙에 저장
    deleteTemp();

    printf("\n");
    return;
}

// 동적 할당 해제
void freeTree(DATA *_r)
{
    if (_r == NULL)
    {
        return;
    }
    freeTree(_r->left);
    freeTree(_r->right);
    free(_r);
}

int main()
{

    // 기준 시간
    int t = 0;
    scanf("%d", &t);
    // 입력 받은 시간이 1보다 작은 경우 종료
    if (t < 1)
    {
        printf("잘못된 시간입니다.\n");
        return 0;
    }
    
    // word_data.txt 파일 열기
    FILE *f = fopen("word_data.txt", "rt"); // read text
    if (f == NULL)
    {
        printf("unable to open file");
        return 0;
    }

    // 문자열과 숫자 분리
    char word[100];
    int time;

    while (1)
    {
        // 단어와 시간 분리
        int element = fscanf(f, "%s %d", word, &time);

        // 읽을 게 없으면 종료
        if (element <= 0)
        {
            break;
        }

        // 단어를 트리와 힙에 추가
        // 이미 있는 단어인 경우 빈도 수 올리고 정렬, 없는 단어인 경우 추가 후 정렬
        addData(word);

        // 빈도수 상위 5개 출력
        printf("t = %d일때 순위\n", time);
        showHot();

        // 시간 도달 시 종료
        if (time == t)
        {
            break;
        }
    }

    // 파일 닫기
    fclose(f);

    // 동적할당 해제
    freeTree(root);
    return 0;
}
```