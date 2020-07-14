### 根据对象的某个属性排序

```java
    /**
     * 根据实体对象某个属性排序
     *
     * @param array    对象数组
     * @param function 属性
     * @param <T>      实体对象
     * @param <R>      属性返回值
     */
    public static <T, R extends Comparable<R>> T[] sort(T[] array, Function<T, R> function) {
        Arrays.stream(array)
                .sorted(Comparator.comparing(function))
                .collect(Collectors.toList()).toArray(array);
        return array;
    }

    @Test
    public void testArraySort() {
        Person[] personArray = new Person[]{
                new Person(3, "c"),
                new Person(2, "b"),
                new Person(1, "a"),
        };

        for (Person person : personArray) {
            System.out.println("before: " + person);
        }

        final Person[] people = sort(personArray, Person::getName);

        for (Person person : people) {
            System.out.println("after:  " + person);
        }
    }

@Data
@AllArgsConstructor
class Person {
    private int id;
    private String name;
}
```

