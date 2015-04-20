title: "Tip for Writing Better NUnit Test Cases"
date: 2009-12-18 10:43:04
tags:
---

If you have method, which does the string reverse. So normally we writes test cases in Nunit something like this

```cs
[Test]
public void TestCase1()
{
  ExString exString = new ExString();
  Assert.AreEqual("dcba", exString.Reverse("abcd"));
}

[Test]
public void TestCase2()
{
  ExString exString = new ExString();
  Assert.AreEqual("abba", exString.Reverse("abba"));
}
```

This means, you are repeating the same code again and again in order to test different possibilities. The down side of this approach is, in long run it will be very difficult to manage the test case when compared to the actual code.

To solve this particular problem NUnit has got parameterised test cases attribute, so the same could be re write above test case to

```cs
[TestCase("abcd", "dcba", TestName = "Test Case 1")]
[TestCase("abcd1", "1dcba", TestName = "Test Case 2")]
[TestCase("aaaa", "aaaa", TestName = "Test Case 3")]
[TestCase("abba", "abba", TestName = "Test Case 4")]
public void Can_Reverse_A_Text(string actual, string expected)
{
  ExString exString = new ExString();
  Assert.AreEqual(expected, exString.Reverse(actual));
}
```

This gives a clean way for testing different possibilities, all these shown in the NUnit test runner as different test cases like below.

![NUnit Screenshot](http://rajeesh.cdn.rhyble.com/images/2009/12/image-thumb.png)

Hope this will help in some way or other
