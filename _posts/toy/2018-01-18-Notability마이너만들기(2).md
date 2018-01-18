---
title: Notability 마이너 만들기(2)
layout: post
category: toy project
---

### 계획 변경
늘어나는 업무로 인해 todo는 계속 늘어나는데 업무때문에 todo마이너를 만들지 못해서 todo 관리는 더 어려워지고 있었다.
그래서 우선 todo 프로그램부터 만들어야겠다고 계획을 변경했다.

### 시작은 간단히
커맨드라인에서 데이터 입출력을 하고 데이터는 H2로 저장하는 간단한 프로그램으로 제작했다.

{% highlight java %}
package com.dinky.todo;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class TodoManager {

    private static Connection c = null;

    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        // DB 연결
        Class.forName("org.h2.Driver");
        String url = "jdbc:h2:mem:~/todo";
        Connection c = DriverManager.getConnection(url);

        // 입력 받은 명령 처리
        Scanner sc = new Scanner(System.in);

        do {
            String command = sc.nextLine();

            switch (command) {
                case "init":
                    initTable();
                    break;
                case "add":
                    String data = sc.nextLine();
                    insertTodo(data);
                    break;
                case "findAll":
                    findAll().forEach(s -> System.out.println(s));
                    break;
                case "q":

            }
        } while (!args[0].equals("q"))

        c.close();
    }

    // 할일 전체 조회
    public static List<Todo> findAll() throws SQLException {
        String select = "SELECT * FROM TODO;";
        PreparedStatement ps = c.prepareStatement(select);
        ResultSet rs = ps.executeQuery();

        List<Todo> todoList = new ArrayList<>();

        while (rs.next()) {
            Todo todo = new Todo();
            todo.setTitle(rs.getString("TITLE"));
            todo.setContent(rs.getString("CONTENT"));
            todoList.add(todo);
        }

        ps.close();
        return todoList;
    }

    // 테이블 생성
    public static void initTable() throws SQLException {
        String createTable = "CREATE TABLE IF EXIST TODO(ID INTEGER, TITLE VARCHAR, CONTENT VARCHAR);";
        PreparedStatement ps = c.prepareStatement(createTable);
        ps.closeOnCompletion();
    }

    // 할일 입력
    public static void insertTodo(String data) throws SQLException {
        String[] dataArr = data.split(" ");
        String id = dataArr[0];
        String title = dataArr[1];
        String content = dataArr[2];

        String insertTodo = "INSERT INTO TODO VALUES (?, ?, ?);";
        PreparedStatement ps = c.prepareStatement(insertTodo);
        ps.setInt(1, id);
        ps.setString(2, title);
        ps.setString(3, content);
        ps.execute();
        ps.close();
    }

}
{% endhighlight %}

### 설계를하자!
의식의 흐름대로 무작정 만들다보니 할일을 입력할때 사용하는 method에서 데이터 검증도 안하고 데이터를 입력 있었다. 그렇다고 검증 로직을 추가하자니 입력외 역할이 추가되는 것이라 괜시리 꺼려지고 리팩토링하고 싶고..

그냥 MVC를 끼얹자!!

### 내 맘속 스프링을 만들자
스프링 내부를 아직 열어보진 않았지만 그동안 사용한 경험을 통해 간단하게 사용할 CLI용 경량 spring과 같은 프레임워크를 만들며 todo를 개선하기로 결정했다.
