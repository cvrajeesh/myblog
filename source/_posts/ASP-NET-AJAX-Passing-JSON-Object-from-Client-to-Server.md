title: "ASP.NET AJAX - Passing JSON Object from Client to Server"
date: 2008-10-27 01:17:58
tags:
- ASP.NET
---

> {% post_link ASP-NET-AJAX-Calling-Server-Side-methods-from-Client-Side "Part 1"%} - Calling Server Side methods from Client Side

In this post, I will explain how to pass data from client to server in JSON format.

Microsoft AJAX client library has built-in methods which helps in serializing and de-serializing JSON. The methods are defined in `Sys.Serialization.JavaScriptSerializer` class. In server side we have `JavaScriptSerializer` that will help you out to do the JSON serialization and de-serialization.

With the help of these two classes, we can easily do the data transfer. Suppose you have class called `Customer` in server side, which looks like this

```cs
namespace ServerClientInteration
{
    [Serializable]
    public class Customer
    {
        public int ID { get; set; }

        public string Name { get; set; }

        public string Address { get; set; }
    }
}
```

This is our payload for this example, we are going to move this object from server to client and vice versa. For that I have exposed two static methods on the server side; one is for retrieving the customer details and the other is for updating the customer.

```cs
  public partial class _Default : System.Web.UI.Page
    {
        private static Customer customer = null;

        protected void Page_Load(object sender, EventArgs e)
        {
            if (customer == null)
            {
                customer = new Customer();
                customer.ID = 1;
                customer.Name = "microsoft";
                customer.Address = "www.microsoft.com";
            }

        }

        [WebMethod]
        [ScriptMethod(UseHttpGet=true)]
        public static string GetTime()
        {
            return DateTime.Now.ToLongTimeString();
        }

        [WebMethod]
        [ScriptMethod(UseHttpGet = true)]
        public static string GetCustomer()
        {
            JavaScriptSerializer serializer = new JavaScriptSerializer();
            return serializer.Serialize(customer);
        }

        [WebMethod]
        [ScriptMethod]
        public static string UpdateCustomer(string jsonCustomer)
        {
            try
            {
                JavaScriptSerializer serializer = new JavaScriptSerializer();
                Customer cust = serializer.Deserialize(jsonCustomer);
                customer.ID = cust.ID;
                customer.Name = cust.Name;
                customer.Address = cust.Address;

                return "Updated";
            }
            catch(Exception Exception)
            {

            }

            return "Error occured while updating";
        }
    }
```

In the above code, you can see `GetCustomer` and `UpdateCustomer` methods. `GetCustomer` will serialize the static customer object and return it as JSON string; `UpdateCustomer` method accepts JSON string as input and that string is de-serialized in to a customer object and the static customer variable is updated with the new values. `UpdateCustomer` will return status message after the update.

Usually, we persist the customer in DB or some other states, but here for simplicity I have used a static variable which will simulate the persistence.

Now we need to call this from the client-side, for that I have two hyperlinks; one will get the customer from the server and other send the customer to the server.

Here is how the HTML looks like

```html
<head runat="server">
  <title>Untitled Page</title>
  <style type="text/css">  
    #result
    {
      width: 250px;
      height: 300px;
      overflow: auto;
      border:1px solid #00d;
    }
  </style>
</head>
<body>
  <form id="form1" runat="server">
    <asp:ScriptManager ID="ScriptManager1" runat="server" EnablePageMethods="true">
    </asp:ScriptManager>
    <div>
        <a href="javascript:callServerMethod()" >Call Server method</a>
        <hr />
        <a href="javascript:getCustomer()" >Get Customer</a>
        <div id="result"></div>
        <hr />
        <div>
          ID : <input type="text" id="custID" /><br />
          Name : <input type="text" id="custName" /><br />
          Adress : <input type="text" id="custAdd" /><br />
        </div>
        <a href="javascript:updateCustomer()" >Update customer</a><br />
    </div>

    <script language="javascript" type="text/javascript">
      //<![CDATA[
      function callServerMethod() {
        PageMethods.GetTime(callBackMethod);
      }

      function callBackMethod(result) {
        alert(result);
      }

      function getCustomer() {
        PageMethods.GetCustomer(callBackGetCustomer);
      }

      function callBackGetCustomer(result) {
        var customer = Sys.Serialization.JavaScriptSerializer.deserialize(result);
        var content = "<br />ID :" +  customer.ID + "<br />" + "Name : " + customer.Name + "<br />" + "Address : " + customer.Address;
        $get("result").innerHTML += content;
      }

      var clientCustomer = function(id,name,add){
        this.ID = id;
        this.Name = name;
        this.Address = add;
      };


      function updateCustomer() {
        var id = $get("custID").value;
        var name = $get("custName").value;
        var address = $get("custAdd").value;

        var customer = new clientCustomer(id,name,address);
        var serializedCustomer = Sys.Serialization.JavaScriptSerializer.serialize(customer);
        PageMethods.UpdateCustomer(serializedCustomer,callBackUpdateCustomer);
      }

      function callBackUpdateCustomer(result) {
        alert(result);
      }

      //]]>
    </script>
  </form>
</body>
```

I think the above code is self explanatory, let me know if you have any difficulty in understanding this. Hope this will gave some idea about passing JSON between client and server.

> I have uploaded the source code in my server and you can get it from here - //static.rajeeshcv.com/download/ServerClientInteration.zip
