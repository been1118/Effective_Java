# 아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩토리와 생성자에는 똑같은 단점이 존재하는데, 선택적 매개변수가 많을 경우 적절히 대응하기가 어렵다.

1. 점층적 생성자 패턴  
   "필수 매개변수"만 받는 생성자부터, "필수 매개변수 + 선택 매개변수" 까지 생성자를 늘려가는 방식이다.
    <details>
    <summary>예시 코드</summary>

   ```java
        public class NutritionFacts {
        
            private final int servingSize;
            private final int servings;
            private final int calories;
            private final int fat;
            private final int sodium;
            private final int carbohydrate;
            
            public NutritionFacts(int servingSize, int servings) {
            this(servingSize, servings, 0);
            }
            
            public NutritionFacts(int servingSize, int servings, int calories) {
            this(servingSize, servings, calories, 0);
            }
            
            public NutritionFacts(int servingSize, int servings, int calories, int fat) {
            this(servingSize, servings, calories, fat, 0);
            }
            
            
            public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
            this(servingSize, servings, calories, fat, sodium, 0);
            }
            
            public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium,
            int carbohydrate) {
            this.servingSize = servingSize;
            this.servings = servings;
            this.calories = calories;
            this.fat = fat;
            this.sodium = sodium;
            this.carbohydrate = carbohydrate;
            }
        }
   ```

    ```
   // 사용
    NutritionFacts nutritionFacts = new NutritionFacts (20, 10, 32,45);
   ```
    </details>

    + 특징
        + 사용자가 설정을 원치 않는 매개변수까지 포함해야하는 경우가 발생할 수 있다.
        + 매개 변수의 순서를 바꿔 보낼 경우 런타임 에러가 나지 않아, 에러 찾기가 매우 어렵다.


2. 자바 빈즈 패턴  
   필드와 메서드를 각각 작성하고, "매개변수가 없는 생성자로 객체" 를 만든 후 "세터 메서드"를 호출하여 매개변수의 값을 설정하는 방식이다.
   <details>
   <summary>예시 코드</summary>
   <div>

   ```java
    public class User {
        private String name;
        private int age;

        public User() {
        }

        public void setName(String name) {
            this.name = name;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }
    ```
   ```java
    public class Example02 {

        User user = new User();
        user.setName("hee");

    }
    ```
   만약 user는 반드시 name 과 age 를 모두 가져야 하는데, name 만 설정한 경우 user 객체는 일관성이 무너진 상태가 된다.
   이때, 수십개를 설정한다면? 큰 오류를 범할 수 있다.
   </div>
   </details>

    + 특징
        + 객체 하나를 만들기 위해서는 여러 개의 메서드를 호출해야 한다.
        + 객체가 완전히 생성되기 전까지 일관성을 보장 받지 못한다.
        + 점층적 생성자 패턴보다는 읽기 쉽니다.


3. 빌더 패턴 (점층적 생성자 패턴의 안전성 + 자바 빈즈 패턴의 가독성)  
   필수 매개변수만으로 생성자 혹은 정적 팩토리를 호출하여 빌더 객체를 얻은 다음, 빌더 객체가 제공하는 세터 메서드들로 원하는 선택 매개변수들을 설정한다.
    <details>
    <summary>예시 코드</summary>
   <div>

    ```java
        public class User {
          private final String name;
          private final int age;
          private final int tall;
          private final String address;

          public static class Builder {
              // 필수
              private final String name;
              private final int age;

              //선택 : 기본 값으로 초기화
              private int tall = 0;
              private String address = "경기도";

              public Builder(String name, int age) {
                  this.name = name;
                  this.age = age;
              }
              public Builder tall(int value) {
                  tall = value;
                  return this;
              }

              public Builder address(String value) {
                  address = value;
                  return this;
              }

              public User build() {
                  return new User(this);
              }
          }

          private User(Builder builder) {
              name = builder.name;
              age = builder.age;
              tall = builder.tall;
              address = builder.address;
          }
      }
   ```
   ```java
      // 사용 
      	User user = new User.Builder("hee", 14)  // 필수
                                     .tall(140)  // 선택
                                     .build();
    ```  
   User는 private final 로 인해 불변 객체가 되었다.
   </div>
   </details>

    + 특징
        + 빌더는 생성할 클래스 안에 주로 정적 멤버 클래스를 만든다.
        + 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓸 대 빛을 발한다 :)


+ 객체를 만들기 위해서는 빌더부터 만들어야한다.
+ 빌더 패턴은 매개변수가 4개 이상은 되어야 쓸만하다.(그러나 종종 API는 시간이 지날 수록 매개변수가 많아진다고 한다. ㅎㅎ)
