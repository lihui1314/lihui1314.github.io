---
layout: post
title: "Binary Tree"
excerpt: "二叉树c 版本"
---
### Binary Tree

#### 声明

```
///tree
struct TreeNode {
    struct TreeNode *left;
    struct TreeNode *right;
    int val;
};

void preOrder(struct TreeNode *root);

void inOrder(struct TreeNode *root);

void postOrder(struct TreeNode *root);

struct TreeNode *creatSearchTree(int *a, int n);

int treeHeight(struct TreeNode *root);

void reverseTree(struct TreeNode *root);

```

#### 创建搜索二叉树

+ 搜索二叉树：右孩子 > 父节点 > 左孩子

```
// 因为只是做测试用， 这个创建的有点暴力，应该还有更好的方法。😂
struct TreeNode *creatTreeNode(int val);

// 搜索插入点
struct TreeNode *btSearch(struct TreeNode *root, int val);

// 创建搜索二叉树
struct TreeNode *creatSearchTree(int *a, int n) {
    struct TreeNode *root = creatTreeNode(a[0]);
    for (int i = 1; i < n; i++) {
        struct TreeNode *td = btSearch(root, a[i]);
        if (td->val >= a[i]) {
            struct TreeNode *left  = creatTreeNode(a[i]);
            left->val = a[i];
            td->left = left;
        } else {
            struct TreeNode *right  = creatTreeNode(a[i]);
            right->val = a[i];
            td->right = right;
        }
    }
    return root;
}

struct TreeNode *creatTreeNode(int val) {
    struct TreeNode *root = malloc(sizeof(struct TreeNode));
    root->left = NULL;
    root->right = NULL;
    root->val = val;
    return root;
}


struct TreeNode *btSearch(struct TreeNode *root, int val) {
    if (root == NULL) {
        return NULL;
    }
    if (root->val >= val  && root->left == NULL) {
        return root;
    } else if (root->val >= val) {
        return btSearch(root->left, val);
    } else if (root->val < val && root->right == NULL) {
        return root;
    } else {
        return btSearch(root->right, val);
    }
}

```



#### 深度优先遍历

```
// 前序
void preOrder(struct TreeNode *root) {
    if (root == NULL) {
        return;
    }
    printf("%d\n",root->val);
    preOrder(root->left);
    preOrder(root->right);
}

//中序
void inOrder(struct TreeNode *root) {
    if (root == NULL) {
        return;
    }
    inOrder(root->left);
    printf("%d\n",root->val);
    inOrder(root->right);
}

//后续
void postOrder(struct TreeNode *root) {
    if (root == NULL) {
        return;
    }
    postOrder(root->left);
    postOrder(root->right);
    printf("%d\n",root->val);
}

```



#### 广度优先遍历

广度优先遍历需要队列辅助

队列：先进先出。

链式队列声明：

```
///  Queue
struct TLNode {
    struct TLNode *next;
    void *data;
};

struct Queue {
    struct TLNode *front;
    struct TLNode *rear;
};

struct Queue *creatQueue(void);
bool queueIsEmpty(struct Queue *q);
void queueAppend (struct Queue *queue, void *);
void queueDelete (struct Queue *queue);
```

链式队列实现：

```
struct Queue *creatQueue(void) {
    struct Queue *q = malloc(sizeof(struct Queue));
    q->front = NULL;
    q->rear = NULL;
    return q;
}

bool queueIsEmpty(struct Queue *q) {
    if (q->front == NULL) {
        return true;
    }
    return false;
}

void queueAppend (struct Queue *queue, struct TreeNode *data) {
    struct TLNode *node = malloc(sizeof(struct TLNode));
    node->next = NULL;
    node->data = data;
    if (queue->front == NULL) {
        queue->front = node;
        queue->rear = node;
    } else {
        queue->rear->next = node;
        queue->rear = node;
    }
}

void queueDelete (struct Queue *queue) {
    if (queueIsEmpty(queue)) {
        return;
    }
    struct TLNode *node = queue->front;
    node->data = NULL;
    queue->front = queue->front->next;
    free(node);
}

```



广度优先遍历二叉树：

```
void bfsTree(struct TreeNode *root) {
    struct Queue *q = creatQueue();
    queueAppend(q, root);
    while (!queueIsEmpty(q)) {
        struct TreeNode *node = q->front->data;
        printf("%d\n",node->val);
        if (node->left != NULL) {
            queueAppend(q, node->left);
        }
        if (node->right != NULL) {
            queueAppend(q, node->right);
        }
        queueDelete(q);
    }
}
```



#### 二叉树反转

+ 因为曾经有一个大佬面试谷歌没写出来火了😂

```
void reverseTree(struct TreeNode *root) {
    if (root == NULL) {
        return;
    }
    struct TreeNode *temp = root->left;
    root->left = root->right;
    root->right = temp;
    
    reverseTree(root->left);
    reverseTree(root->right);
}
```



#### 二叉树深度

```
int treeHeight(struct TreeNode *root) {
    if (root == NULL) {
        return 0;
    }
    if (root->left == NULL && root ->right == NULL) {
        return 1;
    }
    int left = 1 + treeHeight(root->left);
    int right = 1 + treeHeight(root->right);
    return  left > right ? left : right;
}
```



#### 判断平衡二叉树

这里写了我自己的一个写法，不过速度也击败了99.2%，力扣上有更优雅一点的写法😂

```
int ltreeHeight(struct TreeNode* root, int *b);
bool isBalanced(struct TreeNode* root){
    int b = 0;
    ltreeHeight(root,&b);
    if(b == 1) {
        return false;
    }
    return true;
}

int ltreeHeight(struct TreeNode* root, int *b){
     if (root == NULL) {
         return 0;
     }
     if (root->left == NULL && root ->right == NULL) {
         return 1;
     }
     int left = 1 + ltreeHeight(root->left , b);
     int right = 1 + ltreeHeight(root->right, b);
     if(abs(left - right) > 1) {
        *b = 1;
     }
     return  left > right ? left : right;
}
```