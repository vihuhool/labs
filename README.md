# Introduction
## Simple Functions
```
fun start(): String = "OK"
```
## Named arguments
```
fun joinOptions(options: Collection<String>) = 
    options.joinToString(prefix="[",postfix="]")
```
## Default arguments
```
fun foo(name: String, number: Int = 42, toUpperCase: Boolean = false) =
        (if (toUpperCase) name.uppercase() else name) + number

fun useFoo() = listOf(
        foo("a"),
        foo("b", number = 1),
        foo("c", toUpperCase = true),
        foo(name = "d", number = 2, toUpperCase = true)
)
```
## Triple-quoted strings
```
const val question = "life, the universe, and everything"
const val answer = 42

val tripleQuotedString = """
    #question = "$question"
    #answer = $answer""".trimMargin("#")

fun main() {
    println(tripleQuotedString)
}
```
## String templates
```
val month = "(JAN|FEB|MAR|APR|MAY|JUN|JUL|AUG|SEP|OCT|NOV|DEC)"

fun getPattern(): String = """\d{2} $month \d{4}"""
```
## Nullable types
```
fun sendMessageToClient(
        client: Client?, message: String?, mailer: Mailer
) {
    val email = client?.personalInfo?.email
    if (email != null && message != null) {
        mailer.sendMessage(email, message)
    }
}

class Client(val personalInfo: PersonalInfo?)
class PersonalInfo(val email: String?)
interface Mailer {
    fun sendMessage(email: String, message: String)
}
```
## Nothing type
```
import java.lang.IllegalArgumentException

fun failWithWrongAge(age: Int?): Nothing {
    throw IllegalArgumentException("Wrong age: $age")
}

fun checkAge(age: Int?) {
    if (age == null || age !in 0..150) failWithWrongAge(age)
    println("Congrats! Next year you'll be ${age + 1}.")
}

fun main() {
    checkAge(10)
}
```
## Lambdas
```
fun containsEven(collection: Collection<Int>): Boolean =
        collection.any { it % 2 == 0 }
```
# Classes
## Data classes
```
data class Person(val name: String, val age: Int)

fun getPeople(): List<Person> {
    return listOf(Person("Alice", 29), Person("Bob", 31))
}

fun comparePeople(): Boolean {
    val p1 = Person("Alice", 29)
    val p2 = Person("Alice", 29)
    return p1 == p2  // should be true
}
```
## Smart casts
```
fun eval(expr: Expr): Int =
        when (expr) {
            is Num -> expr.value
            is Sum -> eval(expr.left) + eval(expr.right)
            else -> throw IllegalArgumentException("Unknown expression")
        }

interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```
## Sealed classes
```
fun eval(expr: Expr): Int =
        when (expr) {
            is Num -> expr.value
            is Sum -> eval(expr.left) + eval(expr.right)
        }

sealed interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```
## Rename on import
```
import kotlin.random.Random as KRandom
import java.util.Random as JRandom

fun useDifferentRandomClasses(): String {
    return "Kotlin random: " +
            KRandom.nextInt(2) +
            " Java random:" +
            JRandom().nextInt(2) +
            "."
}
```
## Extension functions
```
fun Int.r(): RationalNumber = RationalNumber(this, 1)

fun Pair<Int, Int>.r(): RationalNumber = RationalNumber(first, second)

data class RationalNumber(val numerator: Int, val denominator: Int)
```
# Conventions
## Comparison
```
data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int) : Comparable<MyDate> {
    override fun compareTo(other: MyDate) = when {
        year != other.year -> year - other.year
        month != other.month -> month - other.month
        else -> dayOfMonth - other.dayOfMonth
    }
}

fun test(date1: MyDate, date2: MyDate) {
    // this code should compile:
    println(date1 < date2)
}
```
## Ranges
```
fun checkInRange(date: MyDate, first: MyDate, last: MyDate): Boolean {
    return date in first..last
}
```
## For loop
```
class DateRange(val start: MyDate, val end: MyDate) : Iterable<MyDate> {
    override fun iterator(): Iterator<MyDate> {
        return object : Iterator<MyDate> {
            var current: MyDate = start

            override fun next(): MyDate {
                if (!hasNext()) throw NoSuchElementException()
                val result = current
                current = current.followingDate()
                return result
            }

            override fun hasNext(): Boolean = current <= end
        }
    }
}

fun iterateOverDateRange(firstDate: MyDate, secondDate: MyDate, handler: (MyDate) -> Unit) {
    for (date in firstDate..secondDate) {
        handler(date)
    }
}
```
## Operators overloading
```
import TimeInterval.*

data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int)

// Supported intervals that might be added to dates:
enum class TimeInterval { DAY, WEEK, YEAR }

operator fun MyDate.plus(timeInterval: TimeInterval) =
        addTimeIntervals(timeInterval, 1)

class RepeatedTimeInterval(val timeInterval: TimeInterval, val number: Int)

operator fun TimeInterval.times(number: Int) =
        RepeatedTimeInterval(this, number)

operator fun MyDate.plus(timeIntervals: RepeatedTimeInterval) =
        addTimeIntervals(timeIntervals.timeInterval, timeIntervals.number)

fun task1(today: MyDate): MyDate {
    return today + YEAR + WEEK
}

fun task2(today: MyDate): MyDate {
    return today + YEAR * 2 + WEEK * 3 + DAY * 5
}
```
## Invoke
```
class Invokable {
    var numberOfInvocations: Int = 0
        private set

    operator fun invoke(): Invokable {
        numberOfInvocations++
        return this
    }
}

fun invokeTwice(invokable: Invokable) = invokable()()
```
# Collections
## Introduction
```
fun Shop.getSetOfCustomers(): Set<Customer> =
        customers.toSet()
```
## Sort
```
// Return a list of customers, sorted in the descending by number of orders they have made
fun Shop.getCustomersSortedByOrders(): List<Customer> =
        customers.sortedByDescending { it.orders.size }
```
## Filter; map
```
// Find all the different cities the customers are from
fun Shop.getCustomerCities(): Set<City> =
        customers.map { it.city }.toSet()
![изображение](https://github.com/vihuhool/labs/assets/69204363/599a9811-0b6e-42aa-9508-5dc3866327f9)


// Find the customers living in a given city
fun Shop.getCustomersFrom(city: City): List<Customer> =
        customers.filter { it.city == city }
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/5d5c06ce-a851-4fbe-a692-9b951dd02fd5)

## All, Any, and other predicates
```
// Return true if all customers are from a given city
fun Shop.checkAllCustomersAreFrom(city: City): Boolean =
        customers.all { it.city == city }

// Return true if there is at least one customer from a given city
fun Shop.hasCustomerFrom(city: City): Boolean =
        customers.any { it.city == city }

// Return the number of customers from a given city
fun Shop.countCustomersFrom(city: City): Int =
        customers.count { it.city == city }

// Return a customer who lives in a given city, or null if there is none
fun Shop.findCustomerFrom(city: City): Customer? =
        customers.find { it.city == city }
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/199f9bf9-eeee-466b-b557-48c4d542b848)

## Associate
```
// Build a map from the customer name to the customer
fun Shop.nameToCustomerMap(): Map<String, Customer> =
        customers.associateBy(Customer::name)

// Build a map from the customer to their city
fun Shop.customerToCityMap(): Map<Customer, City> =
        customers.associateWith(Customer::city)

// Build a map from the customer name to their city
fun Shop.customerNameToCityMap(): Map<String, City> =
        customers.associate { it.name to it.city }
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/cf308624-925f-45c5-b131-579d629d5d2c)

## Group By
```
// Build a map that stores the customers living in a given city
fun Shop.groupCustomersByCity(): Map<City, List<Customer>> =
        customers.groupBy { it.city }
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/60bd8b01-a3ae-42cf-a4cf-a9ed026e1742)

## Partition
```
// Return customers who have more undelivered orders than delivered
fun Shop.getCustomersWithMoreUndeliveredOrders(): Set<Customer> = customers.filter {
    val (delivered, undelivered) = it.orders.partition { it.isDelivered }
    undelivered.size > delivered.size
}.toSet()
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/6e9cd8b5-dc57-4a09-81c3-403204187b79)

## FlatMap
```
// Return all products the given customer has ordered
fun Customer.getOrderedProducts(): List<Product> =
        orders.flatMap(Order::products)

// Return all products that were ordered by at least one customer
fun Shop.getOrderedProducts(): Set<Product> =
        customers.flatMap(Customer::getOrderedProducts).toSet()
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/25a0566d-1a4a-4914-bb7c-e4bfceacb791)

## Max min
```
// Return a customer who has placed the maximum amount of orders
fun Shop.getCustomerWithMaxOrders(): Customer? =
        customers.maxByOrNull { it.orders.size }

// Return the most expensive product that has been ordered by the given customer
fun getMostExpensiveProductBy(customer: Customer): Product? =
        customer.orders
                .flatMap(Order::products)
                .maxByOrNull(Product::price)
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/32c84af5-c6be-4f64-8ce2-6d6f22484d62)

## Sum
```
// Return the sum of prices for all the products ordered by a given customer
fun moneySpentBy(customer: Customer): Double =
        customer.orders.flatMap { it.products }.sumOf { it.price }
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/da661efe-5e7d-4afb-b92d-9f2c1dd209c0)

## Fold and reduce
```
fun Shop.getProductsOrderedByAll(): Set<Product> =customers.map(Customer::getOrderedProducts).reduce { orderedByAll, customer ->
        orderedByAll.intersect(customer)
    }

fun Customer.getOrderedProducts(): Set<Product> =
    orders.flatMap(Order::products).toSet()
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/0ef70aa2-7a1e-4f4a-8397-46a3ee6ea3d9)

## Compound tasks
```
// Find the most expensive product among all the delivered products
// ordered by the customer. Use `Order.isDelivered` flag.
fun findMostExpensiveProductBy(customer: Customer): Product? {
    return customer
        .orders
        .filter(Order::isDelivered)
        .flatMap(Order::products)
        .maxByOrNull(Product::price)
}

// Count the amount of times a product was ordered.
// Note that a customer may order the same product several times.
fun Shop.getNumberOfTimesProductWasOrdered(product: Product): Int {
    return customers
            .flatMap(Customer::getOrderedProducts)
            .count { it == product }
}

fun Customer.getOrderedProducts(): List<Product> =
        orders.flatMap(Order::products)
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/644ea404-58e4-4752-8c52-3b479b611044)

## Sequences
```
// Find the most expensive product among all the delivered products
// ordered by the customer. Use `Order.isDelivered` flag.
fun findMostExpensiveProductBy(customer: Customer): Product? {
    return customer
            .orders
            .asSequence()
            .filter(Order::isDelivered)
            .flatMap(Order::products)
            .maxByOrNull(Product::price)
}

// Count the amount of times a product was ordered.
// Note that a customer may order the same product several times.
fun Shop.getNumberOfTimesProductWasOrdered(product: Product): Int {
    return customers
            .asSequence()
            .flatMap(Customer::getOrderedProducts)
            .count { it == product }
}

fun Customer.getOrderedProducts(): Sequence<Product> =
        orders.asSequence().flatMap(Order::products)
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/2d50c2f4-0087-4d18-99e8-bcd3467ff82d)

# Properties
## Properties
```
class PropertyExample() {
    var counter = 0
    var propertyWithCounter: Int? = null
        set(v) {
            field = v
            counter++
        }
}
}
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/abd07473-40cf-4e6f-b0f7-0b1348b9e2a3)

## Lazy property
```
class LazyProperty(val initializer: () -> Int) {
    var value: Int? = null
    val lazy: Int
        get() {
            if (value == null) {
                value = initializer()
            }
            return value!!
        }
}
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/c2ac8da9-ce81-4441-aca5-0e8ba3079297)

## Delegates example
```
class LazyProperty(val initializer: () -> Int) {
    val lazyValue: Int by lazy(initializer)
}
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/3bc12425-af67-4257-862b-491523572ecf)


## Delegates
```
import kotlin.properties.ReadWriteProperty
import kotlin.reflect.KProperty

class D {
    var date: MyDate by EffectiveDate()
}

class EffectiveDate<R> : ReadWriteProperty<R, MyDate> {

    var timeInMillis: Long? = null

    override fun getValue(thisRef: R, property: KProperty<*>): MyDate {
        return timeInMillis!!.toDate()
    }

    override fun setValue(thisRef: R, property: KProperty<*>, value: MyDate) {
        timeInMillis = value.toMillis()
    }
}
}
```
![изображение](https://github.com/vihuhool/labs/assets/69204363/ff1910de-1e83-41b9-a27c-2e1e3709c2b3)
