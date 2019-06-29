title: java并发编程--线程封闭
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-04-08 16:37:00
subtitle:
header-img:
---
#java并发编程--线程封闭

线程封闭有三种方式：Ad-hoc线程封闭、栈封闭、ThreadLocal类；其中Ad-hoc线程封闭一般由程序员自己实现，很少用，暂不介绍。

 - 加粗    `Ctrl + B` 
 - 插入代码    `Ctrl + K`


## 栈封闭

> 将对象封闭在局部变量中，只有局部变量才能访问，保证线程安全。 

下面我们来看一段简单的代码

```
public class Animals {
    Ark ark;
    Species species;
    Gender gender;

    public int loadTheArk(Collection<Animal> candidates) {
        SortedSet<Animal> animals;
        int numPairs = 0;
        Animal candidate = null;

        // 被封闭在局部方法中，不要让他溢出
        animals = new TreeSet<Animal>(new SpeciesGenderComparator());
        animals.addAll(candidates);
        for (Animal a : animals) {
            if (candidate == null || !candidate.isPotentialMate(a))
                candidate = a;
            else {
                ark.load(new AnimalPair(candidate, a));
                ++numPairs;
                candidate = null;
            }
        }
        return numPairs;
    }


    class Animal {
        Species species;
        Gender gender;

        public boolean isPotentialMate(Animal other) {
            return species == other.species && gender != other.gender;
        }
    }

    enum Species {
        AARDVARK, BENGAL_TIGER, CARIBOU, DINGO, ELEPHANT, FROG, GNU, HYENA,
        IGUANA, JAGUAR, KIWI, LEOPARD, MASTADON, NEWT, OCTOPUS,
        PIRANHA, QUETZAL, RHINOCEROS, SALAMANDER, THREE_TOED_SLOTH,
        UNICORN, VIPER, WEREWOLF, XANTHUS_HUMMINBIRD, YAK, ZEBRA
    }

    enum Gender {
        MALE, FEMALE
    }

    class AnimalPair {
        private final Animal one, two;

        public AnimalPair(Animal one, Animal two) {
            this.one = one;
            this.two = two;
        }
    }

    class SpeciesGenderComparator implements Comparator<Animal> {
        public int compare(Animal one, Animal two) {
            int speciesCompare = one.species.compareTo(two.species);
            return (speciesCompare != 0)
                    ? speciesCompare
                    : one.gender.compareTo(two.gender);
        }
    }

    class Ark {
        private final Set<AnimalPair> loadedAnimals = new HashSet<AnimalPair>();

        public void load(AnimalPair pair) {
            loadedAnimals.add(pair);
        }
    }
}
```

animals被封装在一个局部变量treeset中，并将指向该对象的一个引用保存到animals，此时只有一个引用指向animals，这个引用被封闭在局部变量中，从而实现线程封闭。

## ThreadLocal类
每个线程独有，避免变量共享；为每个线程创建一个副本；
###实现原理
定义了一个静态内部类ThreadLocalMap,key存的是threadLocal实例本身，value存的是副本变量；这样就达到了每个线程有单独的副本变量，实现了变量隔离。ThreadLocalMap采用了散列表，不同于HashMap，主要不同之处在于Hash冲突解决方案，hashmap当冲突时采用的是分离链表法，就是采用链表来存储冲突的value；而ThreadLocalMap采用的是开放定址法，当冲突时会再次做hash运算，知道找到空余的数组，这样做事因为threadLocalMap很少有Hash冲突；
####分离链表法：
![这里写图片描述](http://img.blog.csdn.net/20180408164032350?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd21qNzY1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####开放定址法
![这里写图片描述](http://img.blog.csdn.net/20180408164105234?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd21qNzY1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)