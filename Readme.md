`fork of the amazing https://github.com/vpulim/node-soap/ and modified to provide a few useful features immediately needed`

> Need to avoid xsi:nil="true" attribute to be applied to certain elements

> More research needed to identify exact cause and better solution but gist of issue is that an element defined as a complex reference type without a nullable="true" attribute but whose type actually has the nullable attribute set is getting it.

Example:

```xml
<s:element minOccurs="0" maxOccurs="1" name="Messages" type="tns:ArrayOfString"/>

<s:complexType name="ArrayOfString">
  <s:sequence>
	<s:element
	  minOccurs="0"
	  maxOccurs="unbounded"
	  name="string"
	  nillable="true"
	  type="s:string"
	/>
  </s:sequence>
</s:complexType>
```

Is getting returned as:

```xml
<Messages xsi:nil="true" />
```

This results in the calling applications underlying Java libraries validation to fail:

```javascript
ws = createObject("webservice", "urlto?wsdl")
	.verifyAuthentication({})
	.getMessages()
	.getString();
```

```javascript
// Example usage
soap.listen(
  httpServer,
  options:{
	disableNilOnTags: new Set(["tagName"])
  });
```

changes made in wsdl.js

```diff
// add the new option
WSDL.prototype._initializeOptions = function (options) {
  ...
+ this.options.disableNilOnTags = options.disableNilOnTags || new Set();
}
```

```diff
// prevents from applying the xsi:nil="true" attribute on certain tags
WSDL.prototype.objectToXML = function(...) {
  ...
  parts.push(
	[
-         child === null ? ' xsi:nil="true"' : "",
+         child === null && !self.options.disableNilOnTags.has(name) ? ' xsi:nil="true"' : "",
	].join("")
  );
}
```
