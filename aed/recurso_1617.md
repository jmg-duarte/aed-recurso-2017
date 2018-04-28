# AED Recurso 2017

**1 a)**
![AVL 1,8,10,11](https://s14.postimg.org/4b9w02xep/Screenshot_from_2018-01-07_13-21-53.png)

**1 b)**
![AVL 1,3,5,8,10,11](https://s14.postimg.org/dx3ggkbrl/Screenshot_from_2018-01-07_13-33-54.png)

**2)**
```java
Iterator<Entry<Integer,V>> bucketSort(Iterator<Entry<Integer,V>> entries,
int numberOfKeys) {
    // Implementação do algoritmo de ordenação Bucket Sort.
    List<Entry<Integer, V>> bucket = new List[numberOfKeys+1];
    while(entries.hasNext()){
        Entry<Integer, V> e = entries.next();
        Integer i = e.getEntry();
        if(l == null){
            bucket[i] = new LinkedList<Entry<Integer, V>>();
            bucket[i].add(e);
        } else {
            bucket[i].add(e);
        }
    }
    DoublyLinkedList<Entry<Integer, V>> result = new DoublyLinkedList<>();
    for(List l : bucket){
        if(l != null){
            result.append(l);
        }
    }
    return result.iterator();
}
```

**3)**
```java
boolean isBalanced(BSTNode<K,Integer> treeRoot){
    //True se a árvore (nó) treeRoot for equilibrada.
    BSTNode<K, Integer> left = treeRoot.getLeft();
    BSTNode<K, Integer> right = treeRoot.getRight();
    if(treeRoot.isLeaf())
        return true;
    if(left == null)
        return right.isLeaf();
    if(right == null)
        return left.isLeaf();
    if(left.isLeaf() && right.isLeaf())
        return true;
    if(left.isLeaf() && !right.isLeaf())
        return right.getLeft().isLeaf() && right.getRight().isLeaf();
    if(!left.isLeaf() && right.isLeaf())
        return left.getLeft().isLeaf() && left.getRight().isLeaf();
    return isBalanced(left) && isBalanced(right);
}
```

**4)**

Estrutura de Dados | Explicação
------------ | -------------
`List<String> lines` | Uma lista de linhas permite a iteração inteira do texto e o acesso por indíce de forma fácil.
`HashMap<String, Integer>` | Uma tabela de dispersão sob a forma de `<Palavra, Contagem>` permite o acesso rápido ao número de palavras iguais existentes.
`HashMap<String, List<Integer>` | A tabela de dispersão de `<Palavra, "Linhas">` permite saber em que linhas a palavra está presente de forma rápida e fácil, tendo desta forma acesso a todas as ocorrências.

Operação | Explicação | Complexidade Temporal
-------- | ---------- | ---------------------
`void addLine(String line)` | Dado que temos à nossa disposição a lista, basta adicionar mais uma linha à cauda (`O(1)`), temos também de adicionar para cada palavra uma nova ocorrência aos dicionários (`O(1)` correndo `n` vezes, desta forma `O(n)`) | `O(1) + (O(1) + O(1)) * n = O(n)`
`int numberOfOccurences(String word)` | Acesso à tabela de `<Palavra, Contagem>` | `O(1)`
`Iterator<Integer> whereIsWord(String word)` | Acesso à tabela de `<Palavra, "Linhas">` (`O(1)`). Juntando a criação do iterador da lista de linhas (`O(1)`) | `O(1) + O(1) = O(1)`
`String showLine(int lineNumber)` | Acesso direto à lista, se esta for implementada sob a forma de `ArrayList<String>` temos acesso direto por indíce (`O(1)`), se for de `LinkedList<String>` o acesso apesar de por índice implica percorrermos `lineNumber - 1` elementos `O(n)` | `O(1) ou O(n)`

**5)**

TAD | Elementos da TAD
--- | ----------------
Entidade | NIPC, Nome, Morada, Telefone, Email, Subscrições
Curso | ID, Nome
Especialização | ID, Título, Cursos Associados

Estrutura de Dados | Explicação | Notas 
------------------ | ---------- | -----
`HashMap<String, Entidade>` | Permite inserção, remoção e acesso em tempo constante `O(1)`. Usado pelo sistema para gerir utilizadores de forma rápida e fácil | `K::String -> NIPC`
`HashMap<String, Especialização` | Mesmas razões do `HashMap` acima mas aplicado a especializações, usado pelo sistema para gerir as especializações disponíveis | `K::String -> Especialização.ID`
`HashMap<String, List<Curso>>` | Mesmas razões do `HashMap` acima mas aplicado a cursos, usado pela classe Especialização | `K::String -> Curso.ID`
`SortedMap<String, Especialização>` | Apesar de ter complexidade `O(log n)` na maioria dos casos, permite a ordenação por chave, usado pela classe Entidade | `K::String -> Especialização.ID`

Ex. | Explicação | Complexidade Temporal
--- | ---------- | ---------------------  
**a)** | A existência do `NIPC` é verificado (`O(1)`), se existir é criado uma nova entrada para o utilizador (`O(1)` se a `HashTable` não precisar de crescer) | `O(1) + O(1) = O(1)`
**b)** | Verifica-se a existência do `NIPC` (`O(1)`) e se existir é removido do sistema | `O(1)`
**c)** | Verifica-se a existência do `ID` no sistema, e tal como em **a)** se não existir é criado uma nova entrada | `O(1)`
**d)** | Verifica-se a existência do `ID` no sistema, e tal como em **a)** se não existir é criado uma nova entrada | `O(1)`
**e)** | Ambos os ID's são verificados (`NIPC`, `Especialização.ID`) (`O(1)`) e se ambos existirem a operação prossegue. É efetuado um acesso às tabelas de `<NIPC, Entidade>` e `<ID, Especialização>` (`O(1)`) de modo a obter acesso à entidade e especialização. É depois adicionada uma nova entrada na tabela de subscrições da `Entidade` (`O(1)`). | (`O(1) + O(1) + O(log n) = O(log n)`)
**f)** | O acesso à entidade é feito através da `HashTable` (`O(1)`), de modo a listar as subscrições iteramos o `SortedMap` (`O(n)` pois já está ordenada) | `O(1) + O(n) = O(n)`
**g)** | Acesso à especialização feito através do `HashMap` (`O(1)`), depois é iterada a lista de especializações disponíveis (`O(n)`) | `O(1) + O(n) = O(n)`
