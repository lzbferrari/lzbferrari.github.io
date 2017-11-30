title: 哥尼斯堡七桥问题
date: 2017-04-06 02:57:12
tags:
  - algorithm
  - 欧拉
categories: algorithm
---
　　[柯尼斯堡](https://zh.wikipedia.org/zh-hans/%E6%9F%AF%E5%B0%BC%E6%96%AF%E5%A0%A1)，今日俄罗斯加里宁格勒，哲学家[康德](https://zh.wikipedia.org/zh-hans/%E4%BC%8A%E6%9B%BC%E5%8A%AA%E5%B0%94%C2%B7%E5%BA%B7%E5%BE%B7)的故乡。柯尼斯堡曾经是东普鲁士首府，二战之后割让给了俄国。要是希特勒不作死，这块土地现在应该还是德国的。    

　　这座城市被河流分割成了4块陆地。人们为了连接这些陆地，建了7座桥。
![](http://static.lzb.me/blog/%E5%93%A5%E5%B0%BC%E6%96%AF%E5%A0%A1%E4%B8%83%E6%A1%A5%E9%97%AE%E9%A2%98/%E6%A0%BC%E5%B0%BC%E6%96%AF%E5%A0%A1%E4%B8%83%E6%A1%A5%E9%97%AE%E9%A2%98.svg)
　　现在，需要走遍这7座桥。但是，必须遵守以下条件：
* 走过的桥不能走第二次
* 可以多次经过同一块陆地
* 可以以任意陆地为起点
* 不需要回到起点

　　数学家[莱昂哈德·欧拉](https://zh.wikipedia.org/wiki/%E8%90%8A%E6%98%82%E5%93%88%E5%BE%B7%C2%B7%E6%AD%90%E6%8B%89)(Leonhard Euler,1707-1783)已经将这个问题作为一笔画问题解决。这就是[图论](https://zh.wikipedia.org/wiki/%E5%9B%BE%E8%AE%BA)的起源。  

　　首先把七桥简化。用圆圈来表示陆地1、2、3、4，称为“顶点(vertex)”，用顶点之间的连线来表示桥a、b、c、d、e、f、g，为“边(edge)”，这种连接方式就是“<a href="https://zh.wikipedia.org/wiki/%E5%9B%BE_(%E6%95%B0%E5%AD%A6)">图(graph)</a>”。
<center><img src="http://static.lzb.me/blog/%E5%93%A5%E5%B0%BC%E6%96%AF%E5%A0%A1%E4%B8%83%E6%A1%A5%E9%97%AE%E9%A2%98/%E5%93%A5%E5%B0%BC%E6%96%AF%E5%A0%A1%E4%B8%83%E6%A1%A5_%E5%9B%BE.svg" style="text-align:center;width:60%"/></center>
　　然后，顶点所关联的边数，也就是每块陆地连接的桥数，称为该顶点的度数。
* 陆地0的度数3
* 陆地1的度数5
* 陆地2的度数3
* 陆地3的度数3  

　　接每走过一个桥，桥相连的两个顶点的度数就要减一。因为可以从任意起点开始走，所以不关心具体从哪里开始，也不需要关心每个顶点经过几次，只要关注顶点的度数是如何变化的。  
　　下来开始走，出发时，顶点的度数减1，途中经过每一个顶点时，该顶点的度数减2，因为每通过一个顶点，必须有2条边，一条作为入口，一条作为出口。最后到达终点时，终点度数减一。  
　　如果所有的边都走到，那么最后所有顶点的度数都变为0.  
　　当起点和终点相同的时候，图中所有的顶点，都应该是偶数度数。因为起点一开始减1，结束的时候减1，其它顶点，每走过一次减2.  
　　当起点和终点不相同时，必须有2两个顶点时奇数，这两个顶点作为起点和终点，其它顶点是偶数。

　　所以如果按照规则，走过的桥不能走第二次，且必须走完所有的桥，必须让所有的顶点的度数都变为0，需要满足“所有大陆连接的桥的数量都是偶数，或者2个大陆连接的桥的数量是奇数，其余为偶数”。但实际情况是，所有大陆连接的桥的数量都是奇数。  
　　最后，证明此条件下，不能走遍哥尼斯堡七桥。  

　　哥尼斯堡问题中蕴含着一种思维方式。在观察各个顶点的度数时，重点并不在度数本身，而是“数的奇偶性(parity)”。途中每通过一次顶点，就要减少2个度数，所以顶点的度数必须是2的倍数。

　　下面是代码部分。因为计算机没有欧拉那么机智，所以只能一条一条找，穷举所有最长的一笔画路径。  
　　这个图是多重图，可以用[邻接表](https://zh.wikipedia.org/zh-hans/%E9%82%BB%E6%8E%A5%E8%A1%A8)这种数据结构来表达。需要知道顶点的个数，和边的个数，以及每条边连接的是哪两个顶点。  
　　七桥问题中，因为 有4个顶点，7条边。七条边从a-g分别为，0-1,0-1,0-3,1-3,1-2,1-2,2-3.  
　　遍历出所有的不重复的最长路径，[深度优先搜索](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)。
```
package me.lzb.example.graph;

import java.util.*;

/**
 * 遍历无向图的所有最长一笔画
 * 深度优先，达到最深时，后退，继续搜索另一条路径
 * Created by LZB on 2017/4/8.
 */
public class Graph {
    /**
     * 换行符
     */
    private static final String NEWLINE = System.getProperty("line.separator");
    /**
     * 路径分割符号
     */
    private static final String PATH_SEPARATOR = "->";

    /**
     * 顶点数目
     */
    private int vertexCount;

    /**
     * 边的数目
     */
    private int edgeCount;

    /**
     * 出现过路径的最长变数
     * 如果等于总边数，说明存在欧拉路径
     */
    int maxEdge = 0;

    /**
     * 顶点数组，每个list是与顶点关联的所有边
     */
    private LinkedList<Edge>[] edgeList;

    /**
     * 边
     */
    private class Edge {
        /**
         * 边的id
         */
        int id;

        /**
         * 是否被正向搜索
         */
        boolean isSearched;

        /**
         * 顶点v
         */
        int v;

        /**
         * 顶点b
         */
        int w;

        /**
         * 保存回滚操作中，被回滚的的路径方向，以及，前提路径
         * 因为在不同级别的回滚中，可能会有多条临时路径，所以用list存放
         * 顶点->顶点:路径id->路径id->路径id
         * 1->2:0->1->2
         */
        ArrayList<String> to = new ArrayList<>();

        /**
         * 构造函数
         * @param v 顶点v
         * @param w 顶点w
         */
        public Edge(int v, int w) {
            this.v = v;
            this.w = w;
            isSearched = false;
            id = edgeCount;
        }


        /**
         * 在当前前提路径下，是否有
         * @param v0 出发顶点
         * @param P  前提路径
         * @return true false
         */
        public boolean isFrom(int v0, String P) {
            return isTheSameTo(v0, getAnotherV(v0), P);
        }

        /**
         * 临时路径是否相同
         * @param v0 出发顶点
         * @param v1 到达顶点
         * @param p  前提路径
         * @return true false
         */
        public boolean isTheSameTo(int v0, int v1, String p) {
            if (to.size() == 0) {
                return false;
            }
            String ss = v0 + PATH_SEPARATOR + v1 + ":" + p;
            for (String s : to) {
                if (ss.equals(s)) {
                    return true;
                }
            }
            return false;
        }

        /**
         * 删除临时路径
         * @param v0 出发顶点
         * @param v1 到达顶点
         * @param p  前提路径
         */
        public void removeTo(int v0, int v1, String p) {
            if (to.size() == 0) {
                return;
            }
            String ss = v0 + PATH_SEPARATOR + v1 + ":" + p;
            for (Iterator<String> iterator = to.iterator(); iterator.hasNext(); ) {
                String s = iterator.next();
                if (ss.equals(s)) {
                    iterator.remove();
                    return;
                }
            }
        }

        /**
         * 增加临时路径
         * @param v0 出发顶点
         * @param v1 到达顶点
         * @param p  前提路径
         */
        public void addTo(int v0, int v1, String p) {
            String ss = v0 + PATH_SEPARATOR + v1 + ":" + p;
            for (String s : to) {
                if (ss.equals(s)) {
                    return;
                }
            }
            to.add(ss);

        }

        /**
         * 获取边的另外一条顶点
         * @param vertex
         * @return
         */
        public int getAnotherV(int vertex) {
            if (vertex == v) {
                return w;
            } else {
                return v;
            }
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            Edge c = (Edge) obj;
            return this.id == c.id;
        }

        @Override
        public int hashCode() {
            return id;
        }
    }

    /**
     * 构造函数
     * @param vertexNum 顶点总数
     * @param edgeCount 边的总数
     */
    public Graph(int vertexNum, int edgeCount) {
        this.vertexCount = vertexNum;
        this.edgeCount = 0;
        edgeList = new LinkedList[edgeCount];
        for (int i = 0; i < edgeCount; i++) {
            edgeList[i] = new LinkedList<>();
        }
    }

    public void addEdge(int v1, int v2) {
        Edge c = new Edge(v2, v1);
        edgeList[v1].add(c);
        edgeList[v2].add(c);
        edgeCount++;
    }


    public void addEdge(int[][] edgeArray) {
        for (int i = 0; i < edgeArray.length; i++) {
            addEdge(edgeArray[i][0], edgeArray[i][1]);
        }
    }

    public String toString() {
        StringBuilder s = new StringBuilder();
        s.append(vertexCount + " vertices, " + edgeCount + " edges " + NEWLINE);
        for (int v = 0; v < vertexCount; v++) {
            s.append(v + ": ");
            for (Edge w : edgeList[v]) {
                s.append(w.getAnotherV(v) + " ");
            }
            s.append(NEWLINE);
        }
        return s.toString();
    }


    /**
     * 更新出现过路径的最长边数
     * @param a
     */
    private void updateMax(int a) {
        if (a > maxEdge) {
            maxEdge = a;
        }
    }


    public boolean isEuler() {
        int start = 0;
        Stack<Integer> stack = new Stack<>();
        stack.push(start);

        //TODO 退出递归的条件
        try {
            search(start, start, stack, new Stack<>());
        }catch (EmptyStackException e){

        }

        System.out.println("最长边数：" + maxEdge);
        return maxEdge == edgeCount;
    }




    /**
     * 正向搜索
     * 传进去一个节点，顺着一条没有搜索过的边找到下一个节点。当搜索到死路时，回滚
     * @param v 当前提点
     * @param stack 当前路径的节点顺序
     * @param sp 当前路径的路径顺序
     */
    public void search(int start, int v, Stack<Integer> stack, Stack<Edge> sp) {

        LinkedList<Edge> list = edgeList[v];

        boolean anotherWay = false;
        for (Edge w : list) {
            if (!w.isSearched && !w.isTheSameTo(v, w.getAnotherV(v), getPath(sp))) {
                anotherWay = true;
                w.isSearched = true;
                stack.push(w.getAnotherV(v));
                updateMax(sp.size());
                sp.push(w);
                search(start, w.getAnotherV(v), stack, sp);
            }
        }

        if (!anotherWay) {
            System.out.println("最长：===============================");
            rollback(start, stack, sp);
        }

    }



    /**
     * 回滚，回滚当上一个节点，如果当前节点有可以使用的边，调用搜索，如果没有，递归继续回滚
     * 如果需要递归回滚，回滚到第二级之前，清空所有,当前路径下，从该点出发的方向
     * 被回滚的路径，需要保存路径方向，以及提前路径
     * @param stack 当前路径的节点顺序
     * @param sp 当前路径的路径顺序
     */
    public void rollback(int start, Stack<Integer> stack, Stack<Edge> sp) {

        String ss = getPath(sp);
        String output = "顶点：" + stack.toString()
            + NEWLINE + "路径：" + ss
            + NEWLINE;
        System.out.println(output);

        Edge e = sp.pop();  //需要回滚的路径
        String pp = getPath(sp);    //前提路径

        int vz = stack.pop();
        int vy = stack.peek();

        boolean rollbakc2 = true;

        LinkedList<Edge> l = edgeList[vy];

        //判断当前节点是否存在空闲路径，是否要回滚两级
        //空闲路径：没有被正向搜索，也没有被缓存当前前提路径下，从改节点出发的方向
        for (Edge w : l) {
            if (!w.isSearched && !w.isTheSameTo(vy, w.getAnotherV(vy), pp)) {
                rollbakc2 = false;
                break;
            }
        }


        //回滚当前路径，回滚一级
        int r = vy;
        for (Edge w : l) {
            if (w.equals(e)) {
                w.addTo(vy, vz, pp);
                w.isSearched = false;
                break;
            }
        }

        if (rollbakc2) {
            //回滚两级, 清空所有,当前路径下，从该点出发的方向
            for (Edge w : l) {
                if (!w.isSearched && w.isFrom(vy, pp)) {
                    w.removeTo(vy, w.getAnotherV(vy), pp);
                }
            }
            rollback(start, stack, sp);
        }

        search(start, r, stack, sp);

    }

    public String getPath(Stack<Edge> stack) {
        String s = "";
        for (Edge x : stack) {
            s = s + x.id + PATH_SEPARATOR;
        }
        s = s.replaceAll(PATH_SEPARATOR + "$", "");
        return s;
    }


    public static void main(String[] args) {
        int[][] aa = new int[][]{{0, 1}, {0, 1}, {0, 3}, {1, 3}, {1, 2}, {1, 2}, {2, 3}};
        Graph g = new Graph(4, aa.length);
        g.addEdge(aa);
        System.out.println(g.toString());
        System.out.println(g.isEuler());
    }
}

```
