---
layout: post
title:  "Rem's algorithm 의 동작 방법"
date:   2020-09-14 22:06:22 +0900
categories: jekyll update
---

Union-Find algorithm 과 Rem's algorithm의 차이를 알아보자.

### Traditional Union-Find 의 동작 
아래 그림을 통해서 union 메소드가 어떻게 동작하는지 알아보자. 아래 그림의 예시는 union(6, 5)를 수행할 때의 예시이다.<br />
![unionfind](https://github.com/dongchime/dongchime.github.io/blob/master/assets/img/uf.jpg?raw=true) <br />
"union" 연산에서 노드 5와 6에 대해 find 연산을 진행하면서 root까지 올라가기 때문에 노드 5와 6보다 위에 있는 노드들에 대한 find 연산이 이루어진다.

### Rem's algorithm 의 동작
Rem's algorithm 의 "union" 연산의 동작 방법을 보자. 이 예시에서도 위와 같이 union(6, 5)를 수행한다. <br /> 
![unionfind](https://github.com/dongchime/dongchime.github.io/blob/master/assets/img/rem.jpg?raw=true) <br />
Rem's algorithm의 "union" 연산에서는 root까지 find를 해보는 대신 노드 u와 v의 부모노드 p(u)와 p(v)를 비교한 후, p(u) < p(v) 일때는 u가 p(v)에 연결되고, p(u) > p(v) 일때는 v가 p(u)에 연결되는 방식이다. 
그리고 u 혹은 v 가 p(u) 혹은 p(v)로 올라간다. 이 작업을 두 노드 u, v의 부모노드가 같거나, 두 노드 중 한 노드가 부모노드와 같은 값일 때까지 수행한다.  

### Rem's algorithm 의 union 연산 코드 (JAVA)

```bash
public static boolean union(int u, int v){
        int up, vp;

        while((up = p.getOrDefault(u, u)) != (vp = p.getOrDefault(v, v))){
            if(up < vp){
                if(vp == v){
                    p.put(v, up);                  
                    return true;
                }
                p.put(v, up);
                v = vp;
            }
            else{
                if(up == u){
                    p.put(u, vp);
                    return true;
                }
                p.put(u, vp);
                u = up;
            }
        }
        return false;
    }
```