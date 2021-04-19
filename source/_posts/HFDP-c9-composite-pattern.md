---
title: 组合模式
date: 2021-04-19 19:00:00
categories:
- HFDP
tags:
- composite pattern
- 组合模式
---

> **The Composite Pattern** allows you to compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.
> 组合模式让我们可以将数据组合成树状结构。他可以让我们以统一的模式对待单个节点或整个组合体

## 缘起

书接上一个 Iterator 章节，现在 DinerMenu 业务扩展了，我们想要在 DinerMenu 的基础上再增加一个子目录来打印一个点心子菜单。现在已有的代码结构并不能完成我们的需求，我们将使用 组合模式 重构我们的代码。

更具上面这个需求我们可以将需要展示的结构抽象为如下结构

```txt
                                                     +------------+                                                                                  
                                                     | All Menus  |                                                                                  
                                                     +------------+                                                                                  
                                                        ^                                                                                            
                     -----------------------------------|-------------------------------                                                             
                    |                                   |                              |                                                             
                    |                                   |                              |                                                             
           +--------------------+            +------------+                     +------------+                                                       
           | Pancake House Menu |            | Diner Menu |                     | Cafe Menu  |                                                       
           +--------------------+            +------------+                     +------------+                                                       
                    ^                           ^                                  ^                                                                 
     ---------------|----------                 |-------------                     |-----------                                                      
    |               |         |                 |            |                     |          |                                                      
    |               |         |                 |            |                     |          |                                                      
+----------+  +----------+             +----------+   +-------------+    +----------+    +----------+                                                
| MenuItem |  | MenuItem |   ...       | MenuItem |   |Dessert Menu |    | MenuItem |    | MenuItem |                                                
+----------+  +----------+             +----------+   +-------------+    +----------+    +----------+                                                
                                                                ^                                                                                    
                                                  ------------- |--------------                                                                      
                                                 |              |             |                                                                      
                                                 |              |             |                                                                      
                                            +----------+    +----------+      |                                                                      
                                            | MenuItem |    | MenuItem |     ...                                                                     
                                            +----------+    +----------+                                                                             
```

我们要实现用统一的接口访问 Menu 和 MenuItem，所以不难想到，我们需要在这两个概念外面包装一个统一的对外接口。

代码实现如下：

```java
// 我们抽象一个组合节点和叶子结点公用的超集节点，叫做 MenuComponent, 他包含了这两种节点都要用到的方法，虽然这样做会有点冗余
public class MenuComponent {
    public void add(MenuComponent menuComponent) {throw new UnsupportedOperationException();}
    public void remove(MenuComponent menuComponent) {throw new UnsupportedOperationException();}
    public MenuComponent getChild(int i) {throw new UnsupportedOperationException();}
    public String getName() {throw new UnsupportedOperationException();}
    public String getDescription() {throw new UnsupportedOperationException();}
    public double getPrice() {throw new UnsupportedOperationException();}
    public boolean isVegetarian() {throw new UnsupportedOperationException();}
    public void print() {throw new UnsupportedOperationException();}
}

// 叶子结点实现，只需要重写叶子结点支持的方法
public class MenuItem extends MenuComponent {
    String name;
    String description;
    boolean vegetarian;
    double price;

    public MenuItem(String name, String description, boolean vegetarian, double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public double getPrice() {
        return price;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public void print() {
        System.out.print("  " + getName());
        if (isVegetarian()) {
            System.out.print(" (v)");
        }
        System.out.println(", " + getPrice());
        System.out.println("-- " + getDescription());
    }
}

// 组合节点的实现
public class Menu extends MenuComponent {
    ArrayList menuComponents = new ArrayList();
    String name;
    String description;

    public Menu(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public void add(MenuComponent menuComponent) {
        menuComponents.add(menuComponent);
    }

    public void remove(MenuComponent menuComponent) {
        menuComponents.remove(menuComponent);
    }

    public MenuComponent getChild(int i) {
        return (MenuComponent) menuComponents.get(i);
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public void print() {
        System.out.print("\n" + getName());
        System.out.println(", " + getDescription());
        System.out.println("-------------------- -");

        // 打印完自己的信息后，还需要循环答应子节点信息
        Iterator iterator = menuComponents.iterator();
        while (iterator.hasNext()) {
            MenuComponent menuComponent = (MenuComponent) iterator.next();
            menuComponent.print();
        }
    }
}

// 测试的客户端
public class CompositeClient {

    public static void main(String[] args) {
        MenuComponent pancakeHouseMenu = new Menu("PANCAKE HOUSE MENU", "Breakfast");
        MenuComponent dinerMenu = new Menu("DINER MENU", "Lunch");
        MenuComponent cafeMenu = new Menu("CAFE MENU", "Dinner");
        MenuComponent dessertMenu = new Menu("DESSERT MENU", "Dessert of course !");

        MenuComponent allMenus = new Menu("ALL MENUS", "All menus combined");
        allMenus.add(pancakeHouseMenu);
        allMenus.add(dinerMenu);
        allMenus.add(cafeMenu);

        pancakeHouseMenu.add(new MenuItem("K & B’s Pancake Breakfast", "Pancakes with scrambled eggs, and toast", true, 2.99));
        pancakeHouseMenu.add(new MenuItem("Regular Pancake Breakfast", "Pancakes with fried eggs, sausage", false, 2.99));
        pancakeHouseMenu.add(new MenuItem("Blueberry Pancakes", "Pancakes made with fresh blueberries", true, 3.49));
        pancakeHouseMenu.add(new MenuItem("Waffles", "Waffles, with your choice of blueberries or strawberries", true, 3.59));

        dinerMenu.add(new MenuItem("Pasta", "Spaghetti with Marinara Sauce, and a slice of sourdough bread", true, 3.89));
        dinerMenu.add(new MenuItem("Vegetarian BLT", " (Fakin’)Bacon with lettuce & tomato on whole wheat", true, 2.99));
        dinerMenu.add(new MenuItem("BLT", "Bacon with lettuce & tomato on whole wheat", false, 2.99));
        dinerMenu.add(new MenuItem("Soup of the day", "Soup of the day, with a side of potato salad", false, 3.29));

        dinerMenu.add(dessertMenu);
        dessertMenu.add(new MenuItem("Apple Pie", "Apple pie with a flakey crust, topped with vanilla icecream", true, 1.59));

        cafeMenu.add(new MenuItem("Veggie Burger and Air Fries", "Veggie burger on a whole wheat bun, lettuce, tomato, and fries", true, 3.99));
        cafeMenu.add(new MenuItem("Soup of the day", "A cup of the soup of the day, with a side salad", false, 3.69));
        cafeMenu.add(new MenuItem("Burrito", "A large burrito, with whole pinto beans, salsa, guacamole", true, 4.29));
        allMenus.print();
    }
}
// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole
```

上面这种 print 用了内部 Iterator 的方法，简单了很多， 那么如何实现一个外部的 Iterator 呢

我们先为基类添加 createIterator 方法

```java
public class MenuComponent {
    public void add(MenuComponent menuComponent) {throw new UnsupportedOperationException();}
    // dup...
    public Iterator createIterator() {throw new UnsupportedOperationException();};
}
```

然后声明一个 Composite 的 Iterator 的具体实现类, 并让 Composite 实现类返回它

```java
import java.util.Iterator;
import java.util.Stack;

public class CompositeIterator implements Iterator {
    Stack stack = new Stack();

    public CompositeIterator(Iterator iterator) {
        stack.push(iterator);
    }

    @Override
    public boolean hasNext() {
        if (stack.empty()) {
            return false;
        } else {
            Iterator iterator = (Iterator) stack.peek();
            if (!iterator.hasNext()) {
                stack.pop();
                return hasNext();
            } else {
                return true;
            }
        }
    }

    @Override
    public Object next() {
        if (hasNext()) {
            Iterator iterator = (Iterator) stack.peek();
            MenuComponent component = (MenuComponent) iterator.next();
            if (component instanceof Menu) {
                stack.push(component.createIterator());
            }
            return component;
        } else {
            return null;
        }
    }
}

public class Menu extends MenuComponent {
    // dup...
    Iterator iterator = null;
    @Override
    public Iterator createIterator() {
        if (iterator == null) {
            iterator = new CompositeIterator(menuComponents.iterator());
        }
        return iterator;
    }
}
```

由于叶子节点是不需要迭代的，我们返回一个空的 iterator， 每次调用 hasNext() 都返回 false

```java
public class NullIterator implements Iterator {
    @Override
    public boolean hasNext() {
        return false;
    }

    @Override
    public Object next() {
        return null;
    }
}

public class MenuItem extends MenuComponent {
    // dup...
    @Override
    public Iterator createIterator() {
        return new NullIterator();
    }
}
```

测试代码

```java
public class CompositeClient {

    public static void main(String[] args) {
        MenuComponent pancakeHouseMenu = new Menu("PANCAKE HOUSE MENU", "Breakfast");
        MenuComponent dinerMenu = new Menu("DINER MENU", "Lunch");
        MenuComponent cafeMenu = new Menu("CAFE MENU", "Dinner");
        MenuComponent dessertMenu = new Menu("DESSERT MENU", "Dessert of course !");

        MenuComponent allMenus = new Menu("ALL MENUS", "All menus combined");
        allMenus.add(pancakeHouseMenu);
        allMenus.add(dinerMenu);
        allMenus.add(cafeMenu);

        pancakeHouseMenu.add(new MenuItem("K & B’s Pancake Breakfast", "Pancakes with scrambled eggs, and toast", true, 2.99));
        pancakeHouseMenu.add(new MenuItem("Regular Pancake Breakfast", "Pancakes with fried eggs, sausage", false, 2.99));
        pancakeHouseMenu.add(new MenuItem("Blueberry Pancakes", "Pancakes made with fresh blueberries", true, 3.49));
        pancakeHouseMenu.add(new MenuItem("Waffles", "Waffles, with your choice of blueberries or strawberries", true, 3.59));

        dinerMenu.add(new MenuItem("Pasta", "Spaghetti with Marinara Sauce, and a slice of sourdough bread", true, 3.89));
        dinerMenu.add(new MenuItem("Vegetarian BLT", " (Fakin’)Bacon with lettuce & tomato on whole wheat", true, 2.99));
        dinerMenu.add(new MenuItem("BLT", "Bacon with lettuce & tomato on whole wheat", false, 2.99));
        dinerMenu.add(new MenuItem("Soup of the day", "Soup of the day, with a side of potato salad", false, 3.29));

        dinerMenu.add(dessertMenu);
        dessertMenu.add(new MenuItem("Apple Pie", "Apple pie with a flakey crust, topped with vanilla icecream", true, 1.59));

        cafeMenu.add(new MenuItem("Veggie Burger and Air Fries", "Veggie burger on a whole wheat bun, lettuce, tomato, and fries", true, 3.99));
        cafeMenu.add(new MenuItem("Soup of the day", "A cup of the soup of the day, with a side salad", false, 3.69));
        cafeMenu.add(new MenuItem("Burrito", "A large burrito, with whole pinto beans, salsa, guacamole", true, 4.29));

        printVegetarianMenu(allMenus);
    }

    private static void printVegetarianMenu(MenuComponent menuComponent) {
        Iterator iterator = menuComponent.createIterator();
        System.out.println("\nVEGETARIAN MENU \n---");
        while (iterator.hasNext()) {
            MenuComponent component = (MenuComponent) iterator.next();
            try {
                if (component.isVegetarian()) {
                    menuComponent.print();
                }
            } catch (UnsupportedOperationException e) {
            }
        }
    }
}

// VEGETARIAN MENU

// --- ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole

// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole

// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole

// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole

// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole

// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole

// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole

// ALL MENUS, All menus combined
// -------------------- -

// PANCAKE HOUSE MENU, Breakfast
// -------------------- -
//   K & B’s Pancake Breakfast (v), 2.99
// -- Pancakes with scrambled eggs, and toast
//   Regular Pancake Breakfast, 2.99
// -- Pancakes with fried eggs, sausage
//   Blueberry Pancakes (v), 3.49
// -- Pancakes made with fresh blueberries
//   Waffles (v), 3.59
// -- Waffles, with your choice of blueberries or strawberries

// DINER MENU, Lunch
// -------------------- -
//   Pasta (v), 3.89
// -- Spaghetti with Marinara Sauce, and a slice of sourdough bread
//   Vegetarian BLT (v), 2.99
// --  (Fakin’)Bacon with lettuce & tomato on whole wheat
//   BLT, 2.99
// -- Bacon with lettuce & tomato on whole wheat
//   Soup of the day, 3.29
// -- Soup of the day, with a side of potato salad

// DESSERT MENU, Dessert of course !
// -------------------- -
//   Apple Pie (v), 1.59
// -- Apple pie with a flakey crust, topped with vanilla icecream

// CAFE MENU, Dinner
// -------------------- -
//   Veggie Burger and Air Fries (v), 3.99
// -- Veggie burger on a whole wheat bun, lettuce, tomato, and fries
//   Soup of the day, 3.69
// -- A cup of the soup of the day, with a side salad
//   Burrito (v), 4.29
// -- A large burrito, with whole pinto beans, salsa, guacamole
```

输出有很大问题，我回头还要再检查一遍，感觉上这个套路应该没问题才对。

上面用的是外部 Iterator 的方式，需要自己控制当前节点位置，所以实现上比内部的那种要复杂很多。

## UML

图示说明：


```txt
                                                                    
```
