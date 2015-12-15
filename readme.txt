http://www.ibm.com/developerworks/library/ws-whichwsdl/

A WSDL SOAP binding can be either a Remote Procedure Call (RPC) style binding or a document style binding. 
A SOAP binding can also have an encoded use or a literal use. This gives you four style/use models:
1. RPC/encoded
2. RPC/literal
3. Document/encoded
4. Document/literal

1. RPC/encoded WSDL for myMethod
<message name="myMethodRequest">
    <part name="x" type="xsd:int"/>
    <part name="y" type="xsd:float"/>
</message>
<message name="empty"/>

<portType name="PT">
    <operation name="myMethod">
        <input message="myMethodRequest"/>
        <output message="empty"/>
    </operation>
</portType>

RPC/encoded SOAP message for myMethod
<soap:envelope>
    <soap:body>
        <myMethod>
            <x xsi:type="xsd:int">5</x>
            <y xsi:type="xsd:float">5.0</y>
        </myMethod>
    </soap:body>
</soap:envelope>

Strengths
1. The WSDL is about as straightforward as it's possible for WSDL to be.
2. The operation name appears in the message, so the receiver has an easy time dispatching this message to the implementation of the operation.

Weaknesses
1. The type encoding info (such as xsi:type="xsd:int") is usually just overhead which degrades throughput performance.
2. You cannot easily validate this message since only the <x ...>5</x> and <y ...>5.0</y> lines contain things defined in a schema; the rest of the soap:body contents comes from WSDL definitions
3. Although it is legal WSDL, RPC/encoded is not WS-I compliant.



2. RPC/literal WSDL for myMethod
The use in the binding is changed from encoded to literal. That's it.
<message name="myMethodRequest">
    <part name="x" type="xsd:int"/>
    <part name="y" type="xsd:float"/>
</message>
<message name="empty"/>

<portType name="PT">
    <operation name="myMethod">
        <input message="myMethodRequest"/>
        <output message="empty"/>
    </operation>
</portType>

RPC/literal SOAP message for myMethod
<soap:envelope>
    <soap:body>
        <myMethod>
            <x>5</x>
            <y>5.0</y>
        </myMethod>
    </soap:body>
</soap:envelope>

Strengths
1. The WSDL is still about as straightforward as it is possible for WSDL to be.
2. The operation name still appears in the message.
3. The type encoding info is eliminated.
4. RPC/literal is WS-I compliant.

Weaknesses
1. You still cannot easily validate this message since only the <x ...>5</x> and <y ...>5.0</y> lines contain things defined in a schema; the rest of the soap:body contents comes from WSDL definitions.


3. Document/encoded
Nobody follows this style. It is not WS-I compliant. So let's move on.



4. Document/literal
The WSDL for document/literal changes somewhat from the WSDL for RPC/literal
<types>
    <schema>
        <element name="xElement" type="xsd:int"/>
        <element name="yElement" type="xsd:float"/>
    </schema>
</types>

<message name="myMethodRequest">
    <part name="x" element="xElement"/>
    <part name="y" element="yElement"/>
</message>
<message name="empty"/>

<portType name="PT">
    <operation name="myMethod">
        <input message="myMethodRequest"/>
        <output message="empty"/>
    </operation>
</portType>

 Document/literal SOAP message for myMethod
<soap:envelope>
    <soap:body>
        <xElement>5</xElement>
        <yElement>5.0</yElement>
    </soap:body>
</soap:envelope>

Strengths
1. There is no type encoding info.
2. You can finally validate this message with any XML validator. Everything within the soap:body is defined in a schema.
3. Document/literal is WS-I compliant, but with restrictions 

Weaknesses
1. The WSDL is getting a bit more complicated. This is a very minor weakness, however, since WSDL is not meant to be read by humans.
2. The operation name in the SOAP message is lost. Without the name, dispatching can be difficult, and sometimes impossible.
3. WS-I only allows one child of the soap:body in a SOAP message but here soap:body has two children


5. Document/literal wrapped
<types>
    <schema>
        <element name="myMethod">
            <complexType>
                <sequence>
                    <element name="x" type="xsd:int"/>
                    <element name="y" type="xsd:float"/>
                </sequence>
            </complexType>
        </element>
        <element name="myMethodResponse">
            <complexType/>
        </element>
    </schema>
</types>
<message name="myMethodRequest">
    <part name="parameters" element="myMethod"/>
</message>
<message name="empty">
    <part name="parameters" element="myMethodResponse"/>
</message>

<portType name="PT">
    <operation name="myMethod">
        <input message="myMethodRequest"/>
        <output message="empty"/>
    </operation>
</portType>

Document/literal wrapped SOAP message for myMethod
<soap:envelope>
    <soap:body>
        <myMethod>
            <x>5</x>
            <y>5.0</y>
        </myMethod>
    </soap:body>
</soap:envelope>

Strengths
1. There is no type encoding info.
2. Everything that appears in the soap:body is defined by the schema, so you can easily validate this message.
3. Once again, you have the method name in the SOAP message.
4. Document/literal is WS-I compliant, and the wrapped pattern meets the WS-I restriction that the SOAP message's soap:body has only one child.

Weaknesses
1. The WSDL is even more complicated.