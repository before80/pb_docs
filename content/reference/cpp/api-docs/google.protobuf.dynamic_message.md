

+++
title = "dynamic_message.h"
toc_hide = "true"
linkTitle = "C++"
description = "This section contains reference documentation for working with protocol buffer classes in C++."
type = "docs"
+++

<p><code>#include &lt;google/protobuf/dynamic_message.h&gt;<br>namespace <a href="#google.protobuf">google::protobuf</a></code></p><p>Defines an implementation of <a href='google.protobuf.message#Message'>Message</a> which can emulate types which are not known at compile-time. </p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">Classes in this file</h3></th></tr><tr><td><div><code><a href="#DynamicMessageFactory">DynamicMessageFactory</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Constructs implementations of <a href='google.protobuf.message#Message'>Message</a> which can emulate types which are not known at compile-time. </div></td></tr><tr><td><div><code><a href="#DynamicMapSorter">DynamicMapSorter</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Helper for computing a sorted list of map entries via reflection. </div></td></tr></table><h2 id="DynamicMessageFactory">class DynamicMessageFactory: public <a href="google.protobuf.message#MessageFactory">MessageFactory</a></h2><p><code>#include &lt;<a href="#">google/protobuf/dynamic_message.h</a>&gt;<br>namespace <a href="#google.protobuf">google::protobuf</a></code></p><p>Constructs implementations of <a href='google.protobuf.message#Message'>Message</a> which can emulate types which are not known at compile-time. </p><p>Sometimes you want to be able to manipulate protocol types that you don't know about at compile time. It would be nice to be able to construct a <a href='google.protobuf.message#Message'>Message</a> object which implements the message type given by any arbitrary <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a>. DynamicMessage provides this.</p>

<p>As it turns out, a DynamicMessage needs to construct extra information about its type in order to operate. Most of this information can be shared between all DynamicMessages of the same type. But, caching this information in some sort of global map would be a bad idea, since the cached information for a particular descriptor could outlive the descriptor itself. To avoid this problem, <a href='#DynamicMessageFactory'>DynamicMessageFactory</a> encapsulates this "cache". All DynamicMessages of the same type created from the same factory will share the same support data. Any Descriptors used with a particular factory must outlive the factory. </p>

<table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="DynamicMessageFactory.DynamicMessageFactory"><div style="padding-left: 16px; text-indent: -16px"><code><b>DynamicMessageFactory</b>()</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Construct a <a href='#DynamicMessageFactory'>DynamicMessageFactory</a> that will search for extensions in the <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a> in which the extendee is defined. </div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="DynamicMessageFactory.DynamicMessageFactory"><div style="padding-left: 16px; text-indent: -16px"><code><b>DynamicMessageFactory</b>(const <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a> * pool)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Construct a <a href='#DynamicMessageFactory'>DynamicMessageFactory</a> that will search for extensions in the given <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a>.  <a href="#DynamicMessageFactory.DynamicMessageFactory.details">more...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="DynamicMessageFactory.~DynamicMessageFactory"><div style="padding-left: 16px; text-indent: -16px"><code><b>~DynamicMessageFactory</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="DynamicMessageFactory.SetDelegateToGeneratedFactory"><div style="padding-left: 16px; text-indent: -16px"><code><b>SetDelegateToGeneratedFactory</b>(bool enable)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Call this to tell the <a href='#DynamicMessageFactory'>DynamicMessageFactory</a> that if it is given a <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> d for which:  <a href="#DynamicMessageFactory.SetDelegateToGeneratedFactory.details">more...</a></div></td></tr><tr><th colspan="2"><h3 style="margin-top: 4px; margin-bottom: 4px;">implements <a href='google.protobuf.message#MessageFactory'>MessageFactory</a></h3><div style="font-style: italic; font-weight: normal;"></div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual const <a href='google.protobuf.message#Message'>Message</a> *</code></td><td style="border-left-width: 0px"id="DynamicMessageFactory.GetPrototype"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetPrototype</b>(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * type)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">Given a <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a>, constructs the default (prototype) <a href='google.protobuf.message#Message'>Message</a> of that type.  <a href="#DynamicMessageFactory.GetPrototype.details">more...</a></div></td></tr></table> <hr><h3 id="DynamicMessageFactory.DynamicMessageFactory.details"><code> DynamicMessageFactory::DynamicMessageFactory(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a> * pool)</code></h3><div style="margin-left: 16px"><p>Construct a <a href='#DynamicMessageFactory'>DynamicMessageFactory</a> that will search for extensions in the given <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a>. </p><p>DEPRECATED: Use CodedInputStream::SetExtensionRegistry() to tell the parser to look for extensions in an alternate pool. However, note that this is almost never what you want to do. Almost all users should use the zero-arg constructor. </p>
</div> <hr><h3 id="DynamicMessageFactory.SetDelegateToGeneratedFactory.details"><code>void DynamicMessageFactory::SetDelegateToGeneratedFactory(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool enable)</code></h3><div style="margin-left: 16px"><p>Call this to tell the <a href='#DynamicMessageFactory'>DynamicMessageFactory</a> that if it is given a <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> d for which: </p><pre>d-&gt;file()-&gt;pool() == DescriptorPool::generated_pool(),</pre>
<p> then it should delegate to <a href='google.protobuf.message#MessageFactory.generated_factory'>MessageFactory::generated_factory()</a> instead of constructing a dynamic implementation of the message. In theory there is no down side to doing this, so it may become the default in the future. </p>
</div> <hr><h3 id="DynamicMessageFactory.GetPrototype.details"><code>virtual const <a href='google.protobuf.message#Message'>Message</a> * DynamicMessageFactory::GetPrototype(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * type)</code></h3><div style="margin-left: 16px"><p>Given a <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a>, constructs the default (prototype) <a href='google.protobuf.message#Message'>Message</a> of that type. </p><p>You can then call that message's New() method to construct a mutable message of that type.</p>
<p>Calling this method twice with the same <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> returns the same object. The returned object remains property of the factory and will be destroyed when the factory is destroyed. Also, any objects created by calling the prototype's New() method share some data with the prototype, so these must be destroyed before the <a href='#DynamicMessageFactory'>DynamicMessageFactory</a> is destroyed.</p>
<p>The given descriptor must outlive the returned message, and hence must outlive the <a href='#DynamicMessageFactory'>DynamicMessageFactory</a>.</p>
<p>The method is thread-safe. </p>
</div><h2 id="DynamicMapSorter">class DynamicMapSorter</h2><p><code>#include &lt;<a href="#">google/protobuf/dynamic_message.h</a>&gt;<br>namespace <a href="#google.protobuf">google::protobuf</a></code></p><p>Helper for computing a sorted list of map entries via reflection. </p><table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>static std::vector&lt; const <a href='google.protobuf.message#Message'>Message</a> * &gt;</code></td><td style="border-left-width: 0px"id="DynamicMapSorter.Sort"><div style="padding-left: 16px; text-indent: -16px"><code><b>Sort</b>(const <a href='google.protobuf.message#Message'>Message</a> &amp; message, int map_size, const <a href='google.protobuf.message#Reflection'>Reflection</a> * reflection, const <a href='google.protobuf.descriptor#FieldDescriptor'>FieldDescriptor</a> * field)</code></div></td></tr></table>