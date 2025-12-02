```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Copyright 2024 UCAIug

  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

  See the License for the specific language governing permissions and
  limitations under the License.
-->
<xsl:stylesheet exclude-result-prefixes="a" version="3.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:a="http://langdale.com.au/2005/Message#" xmlns="http://langdale.com.au/2009/Indent" xmlns:map="http://www.w3.org/2005/xpath-functions/map" xmlns:fn="http://www.ucaiug.org/functions">

    <xsl:output xmlns:xalan="http://xml.apache.org/xslt" method="xml" omit-xml-declaration="no" indent="yes" encoding="utf-8" xalan:indent-amount="4" />
    <xsl:param name="copyright-single-line" />
	<xsl:param name="version"/>
	<xsl:param name="baseURI"/>
	<xsl:param name="envelope">Profile</xsl:param>
	<xsl:variable name="package_prefix" select="concat(fn:baseuri-to-package($baseURI), '.', lower-case(replace($envelope, '- ', '_')))"/>
	<!-- The following <xsl:text> element is our newline representation where needed. -->
    <xsl:variable name="newline"><xsl:text>
</xsl:text></xsl:variable>
	
	<!-- 
		Function: fn:baseuri-to-package
		Purpose: Converts a baseURI into a package name by reversing the order of dot-separated tokens in a string
		Parameters: 
			$baseURI - The baseURI to reverse (e.g., "a.b.c" becomes "c.b.a")
		Returns: xs:string - The reversed text
		Example: fn:convert-baseuri-to-package('https://ap-voc.cim4.eu/SecurityAnalysisResult/2.4#') returns 'eu.cim4.ap-voc'
	-->
	<xsl:function name="fn:baseuri-to-package" as="xs:string">
		<xsl:param name="text" as="xs:string"/>
		<xsl:variable name="package-text" as="xs:string">
			<xsl:choose>
				<xsl:when test="contains($text, '://')">
					<xsl:value-of select="substring-before(substring-after(lower-case(replace($text, '-', '_')),'://'),'/')"/>
				</xsl:when>
				<xsl:otherwise>
					<xsl:value-of select="$text"/>
				</xsl:otherwise>
			</xsl:choose>
		</xsl:variable>
		
		<xsl:choose>
			<xsl:when test="contains($text, '.')">
				<!-- Recursive call for the part after the first dot -->
				<xsl:variable name="reversed-tail" select="fn:baseuri-to-package(substring-after($package-text, '.'))"/>
				<!-- Concatenate: reversed tail + dot + first token -->
				<xsl:sequence select="concat($reversed-tail, '.', substring-before($package-text, '.'))"/>
			</xsl:when>
			<xsl:otherwise>
				<!-- Base case: no more dots, return the text as-is -->
				<xsl:sequence select="$package-text"/>
			</xsl:otherwise>
		</xsl:choose>
	</xsl:function>

	<!-- 
		Function: fn:fully-qualified-class
		Purpose: Generates a fully qualified class name with package prefix
		Parameters: 
			$element - The element (a:Root or other) to generate the fully qualified class name from
			$package-prefix - The package prefix to prepend to the package/class name
		Returns: xs:string - The fully qualified class name
	-->
	<xsl:function name="fn:fully-qualified-class" as="xs:string">
		<xsl:param name="element" as="element()"/>
		<xsl:param name="package-prefix" as="xs:string"/>
		
		<!-- Determine the class name based on element type -->
		<xsl:variable name="class-name" select="
			if ($element/self::a:Root)
			then 
				concat(
					upper-case(substring(substring-after($element/@baseClass, '#'), 1, 1)),
					substring(substring-after($element/@baseClass, '#'), 2)
				)
			else 
				concat(
					upper-case(substring($element/@name, 1, 1)),
					substring($element/@name, 2)
				)
		"/>
		
		<!-- Build the fully qualified name -->
		<xsl:sequence select="
			if ($element/@package)
			then concat($package-prefix, '.', lower-case($element/@package), '.', $class-name)
			else concat($package-prefix, '.', $class-name)
		"/>
	</xsl:function>
	
	<!-- 
		Function: fn:package-name
		Purpose: Generates a package name based on the element's @package attribute
		Parameters: 
			$element - The element to get the package name from
			$package-prefix - The prefix to prepend to the package name
		Returns: xs:string - The full package name
	-->
	<xsl:function name="fn:package-name" as="xs:string">
		<xsl:param name="element" as="element()"/>
		<xsl:param name="package-prefix" as="xs:string"/>
		
		<xsl:sequence select="
			if ($element/@package)
			then concat($package-prefix, '.', lower-case($element/@package))
			else $package-prefix
		"/>
	</xsl:function>
	
	<!-- 
		Function: fn:to-avro-type
		Purpose: Maps XSD types to Apache Avro primitive types or logical types
		Parameters: 
			$xstype - The XSD type name to map
		Returns: xs:string - The Avro type representation (as a JSON string)
		
		Note: int = 32-bit, long = 64-bit; prefer long for timestamps and ids.
		
		Special cases:
		- date: Uses custom logicalType  (i.e. {"type": "int", "logicalType": "date"} )
		- time: Uses custom logicalType  (i.e. {"type": "int", "logicalType": "time-millis"} )
		- dateTime: Uses custom logicalType  (i.e. {"type": "long", "logicalType": "timestamp-millis"} )
		- duration: Uses custom logicalType (i.e. {"type": "string", "logicalType": "iso8601-duration"} ) "iso8601-duration" (not official Avro)
		  because Avro has no built-in duration type (months/years vary in length)
		- gMonthDay: Uses a record structure with month and day fields  (i.e. {"type": {"type": "record", "name": "MonthDay", "fields": [{"name": "month", "type": "int"},{"name": "day", "type": "int"}]}} )
		  because Avro has no built-in equivalent
	-->
	<xsl:function name="fn:get-avro-type" as="xs:string">
		<xsl:param name="xstype" as="xs:string"/>
		
		<xsl:sequence select="
			if ($xstype = 'string') then '&quot;string&quot;'
			else if ($xstype = 'normalizedString') then '&quot;string&quot;'
			else if ($xstype = 'token') then '&quot;string&quot;'
			else if ($xstype = 'NMTOKEN') then '&quot;string&quot;'
			else if ($xstype = 'anyURI') then '&quot;string&quot;'
			else if ($xstype = 'NCName') then '&quot;string&quot;'
			else if ($xstype = 'Name') then '&quot;string&quot;'
			else if ($xstype = ('integer', 'int', 'short')) then '&quot;int&quot;'
			else if ($xstype = 'long') then '&quot;long&quot;'
			else if ($xstype = 'float') then '&quot;float&quot;'
			else if ($xstype = ('double', 'decimal', 'number')) then '&quot;double&quot;'
			else if ($xstype = 'boolean') then '&quot;boolean&quot;'
			else if ($xstype = 'dateTime') then '{&quot;type&quot;: &quot;long&quot;, &quot;logicalType&quot;: &quot;timestamp-millis&quot;}'
			else if ($xstype = 'date') then '{&quot;type&quot;: &quot;int&quot;, &quot;logicalType&quot;: &quot;date&quot;}'
			else if ($xstype = 'time') then '{&quot;type&quot;: &quot;int&quot;, &quot;logicalType&quot;: &quot;time-millis&quot;}'
			else if ($xstype = 'duration') then '{&quot;type&quot;: &quot;string&quot;, &quot;logicalType&quot;: &quot;iso8601-duration&quot;}'
			else if ($xstype = 'gMonthDay') then '{&quot;type&quot;: {&quot;type&quot;: &quot;record&quot;, &quot;name&quot;: &quot;MonthDay&quot;, &quot;fields&quot;: [{&quot;name&quot;: &quot;month&quot;, &quot;type&quot;: &quot;int&quot;},{&quot;name&quot;: &quot;day&quot;, &quot;type&quot;: &quot;int&quot;}]}}'
			else $xstype
		"/>
	</xsl:function>

	<!-- Single parameter version (starts count at 0) -->
	<xsl:function name="fn:count-fields" as="xs:integer">
		<xsl:param name="element" as="element()"/>
		<xsl:sequence select="fn:count-fields($element, 0)"/>
	</xsl:function>
	
	<!-- Two parameter version (for recursion) -->
	<xsl:function name="fn:count-fields" as="xs:integer">
		<xsl:param name="element" as="element()"/>
		<xsl:param name="count" as="xs:integer"/>
		
		<xsl:variable name="total-count" select="
			$count + count($element/(a:Complex|a:Enumerated|a:Compound|a:SimpleEnumerated|a:Simple|a:Domain|a:Instance|a:Reference|a:Choice))
		"/>
		
		<xsl:sequence select="
			if ($element/a:SuperType) then
				let $baseClass := $element/a:SuperType/@baseClass,
					$parent := root($element)/*/node()[@baseClass = $baseClass]
				return 
					if ($parent) 
					then fn:count-fields($parent, $total-count)
					else $total-count
			else
				$total-count
		"/>
	</xsl:function>
	
	<!-- 
		Function: fn:inherits-from
		Purpose: Checks if an element inherits from a given baseClass (recursively checks SuperType hierarchy)
		Parameters: 
			$element - The element to check (Root or ComplexType)
			$targetBaseClass - The baseClass to check for in the inheritance hierarchy
		Returns: xs:boolean - true if element inherits from targetBaseClass, false otherwise
	-->
	<xsl:function name="fn:inherits-from" as="xs:boolean">
		<xsl:param name="element" as="element()"/>
		<xsl:param name="targetBaseClass" as="xs:string"/>
		
		<xsl:sequence select="
			if ($element/@baseClass = $targetBaseClass) then
				(: Direct match - this element has the target baseClass :)
				true()
			else if ($element/a:SuperType) then
				(: Has a parent - check if parent inherits from target :)
				let $superBaseClass := $element/a:SuperType/@baseClass,
					$parent := (root($element)//a:Root[@baseClass = $superBaseClass]|root($element)//a:ComplexType[@baseClass = $superBaseClass])[1]
				return 
					if ($parent) 
					then fn:inherits-from($parent, $targetBaseClass)
					else false()
			else
				(: No match and no parent - doesn't inherit :)
				false()
		"/>
	</xsl:function>
	
	<!-- 
		Function: fn:model-reference
		Purpose: Gets the model reference for an element (dataType, baseClass, or baseProperty)
		Parameters: 
			$element - The element to get the model reference from
		Returns: xs:string - The model reference value, or empty string if none exists
		
		Rules:
		- SimpleType: uses @dataType
		- EnumeratedType/CompoundType/ComplexType/Root: uses @baseClass
		- Other elements (attributes/associations): uses @baseProperty (if present)
	-->
	<xsl:function name="fn:model-reference" as="xs:string">
		<xsl:param name="element" as="element()"/>
		
		<xsl:sequence select="
			if ($element/self::a:SimpleType) then
				string($element/@dataType)
			else if ($element/self::a:EnumeratedType or 
					 $element/self::a:CompoundType or 
					 $element/self::a:ComplexType or 
					 $element/self::a:Root) then
				string($element/@baseClass)
			else if ($element/@baseProperty) then
				string($element/@baseProperty)
			else
				''
		"/>
	</xsl:function>
	
	<!-- 
		Function: fn:get-union-dependencies
		Purpose: Gets all concrete subclasses (Root elements) that inherit from an abstract class (ComplexType)
				 These represent the union members in Avro schema.
		Parameters: 
			$abstract-element - The ComplexType element (abstract class)
		Returns: xs:string* - Sequence of baseClass values for all concrete subclasses
	-->
	<xsl:function name="fn:get-union-dependencies" as="xs:string*">
		<xsl:param name="abstract-element" as="element()"/>
		
		<xsl:variable name="abstract-baseClass" select="string($abstract-element/@baseClass)"/>
		
		<xsl:sequence select="
			distinct-values(
				for $root in root($abstract-element)//a:Root
				return
					if (fn:inherits-from($root, $abstract-baseClass))
					then string($root/@baseClass)
					else ()
			)
		"/>
	</xsl:function>

	<!-- 
		Function: fn:get-dependencies
		Purpose: Gets all distinct @baseClass values from child elements, 
				 including inherited dependencies from SuperType hierarchy.
				 For Instance/Reference to abstract classes (ComplexType), includes union members.
				 Excludes a:SuperType itself and any types in the exclusion map.
		Parameters: 
			$element - The element to examine (CompoundType, Root, etc.)
			$exclusion-map - Map where keys are baseClass values to exclude
		Returns: xs:string* - Sequence of baseClass values (filtered)
	-->
	<xsl:function name="fn:get-dependencies" as="xs:string*">
		<xsl:param name="element" as="element()"/>
		<xsl:param name="exclusion-map" as="map(xs:string, xs:boolean)"/>
		
		<xsl:sequence select="
			distinct-values(
				(
					(: Dependencies from non-Instance/Reference/InverseInstance/InverseReference children. These we need to exclude. :)
					$element/*[@baseClass][not(self::a:SuperType)][not(self::a:Instance)][not(self::a:Reference)][not(self::a:InverseInstance)][not(self::a:InverseReference)]/string(@baseClass)[not(map:contains($exclusion-map, .))],
					
					(: Dependencies from Instance/Reference children - handle unions :)
					for $assoc in $element/(a:Instance|a:Reference)[@baseClass]
					return
						let $assoc-baseClass := string($assoc/@baseClass),
							$referenced-element := root($element)/*/node()[@baseClass = $assoc-baseClass]
						return
							if ($referenced-element/self::a:ComplexType) then
								(: Abstract class - get union members (concrete subclasses) :)
								fn:get-union-dependencies($referenced-element)[not(map:contains($exclusion-map, .))]
							else if ($referenced-element/self::a:Root) then
								(: Concrete class - use directly :)
								if (not(map:contains($exclusion-map, $assoc-baseClass))) then
									$assoc-baseClass
								else
									()
							else
								(: Referenced element not found or other type - include if not excluded :)
								if (not(map:contains($exclusion-map, $assoc-baseClass))) then
									$assoc-baseClass
								else
									(),
					
					(: If has SuperType, recursively get parent's dependencies :)
					if ($element/a:SuperType) then
						let $supertype_baseClass := $element/a:SuperType/@baseClass,
							$parent := root($element)/*/node()[@baseClass = $supertype_baseClass]
						return
							if ($parent) then
								fn:get-dependencies($parent, $exclusion-map)
							else
								()
					else
						()
				)
			)
		"/>
	</xsl:function>

    <!-- 
        Function: fn:build-dependencies-map
        Purpose: Creates a map of type dependencies for the provided elements
        Parameters: 
            $elements - Node-set of elements to process
            $exclusion-map - Map of baseClass values to exclude from dependencies
        Returns: map(xs:string, xs:string*)
                 - Key: baseClass of each element
                 - Value: Sequence of baseClass values from child elements (filtered)
    -->
    <xsl:function name="fn:build-dependencies-map" as="map(xs:string, xs:string*)">
        <xsl:param name="elements" as="element()*"/>
        <xsl:param name="exclusion-map" as="map(xs:string, xs:boolean)"/>
        
        <xsl:sequence select="
            map:merge(
                for $type in $elements[@baseClass]
                return map:entry(
                    string($type/@baseClass),
                    fn:get-dependencies($type, $exclusion-map)
                )
            )
        "/>
    </xsl:function>
    
	<!-- 
		Function: fn:create-exclusions-map
		Purpose: Creates a map for excluding types based on their @baseClass attribute
		Parameters: 
			$elements - Node-set of elements whose baseClass should be excluded
		Returns: map(xs:string, xs:boolean)
				 - Key: baseClass attribute value
				 - Value: true()
	-->
	<xsl:function name="fn:create-exclusions-map" as="map(xs:string, xs:boolean)">
		<xsl:param name="elements" as="element()*"/>
		
		<xsl:sequence select="
			map:merge(
				for $element in $elements[@baseClass]
				return map:entry(string($element/@baseClass), true())
			)
		"/>
	</xsl:function>
	
	<!-- 
		Function: fn:is-abstract-with-subclasses
		Purpose: Checks if a baseClass references an abstract class (ComplexType) that has concrete subclasses (Roots)
		Parameters: 
			$baseClass - The baseClass URI to check
		Returns: xs:boolean - true if it's an abstract class with concrete subclasses, false otherwise
	-->
	<xsl:function name="fn:is-abstract-with-subclasses" as="xs:boolean">
		<xsl:param name="baseClass" as="xs:string"/>
		
		<xsl:variable name="is-complex-type" select="exists(root()//a:ComplexType[@baseClass = $baseClass])"/>
		
		<xsl:sequence select="
			if ($is-complex-type) then
				(: Count concrete subclasses :)
				let $concrete-count := count(
					for $root in root()//a:Root
					return
						if (fn:inherits-from($root, $baseClass))
						then $root
						else ()
				)
				return $concrete-count > 0
			else
				false()
		"/>
	</xsl:function>

	<!-- 
		Function: fn:topological-sort
		Purpose: Sorts elements in topological order based on their dependencies.
				 Elements with no dependencies come first, then elements that depend only on 
				 already-processed elements, and so on.
				 For circular dependencies, falls back to document order.
		Parameters: 
			$elements - The elements to sort (e.g., //a:CompoundType or //a:Root)
			$deps-map - Dependency map created by fn:build-dependencies-map
		Returns: element()* - Sequence of elements in topological (dependency) order
	-->
	<xsl:function name="fn:topological-sort" as="element()*">
		<xsl:param name="elements" as="element()*"/>
		<xsl:param name="deps-map" as="map(xs:string, xs:string*)"/>
		
		<xsl:sequence select="fn:topological-sort-helper($elements, $deps-map, ())"/>
	</xsl:function>

	<!-- 
		Function: fn:topological-sort-helper
		Purpose: Recursive helper for topological sort with accumulated results
		Parameters: 
			$remaining - Elements not yet sorted
			$deps-map - Dependency map
			$sorted - Accumulated sorted elements
		Returns: element()* - Fully sorted sequence
	-->
	<xsl:function name="fn:topological-sort-helper" as="element()*">
		<xsl:param name="remaining" as="element()*"/>
		<xsl:param name="deps-map" as="map(xs:string, xs:string*)"/>
		<xsl:param name="sorted" as="element()*"/>
		
		<xsl:choose>
			<xsl:when test="empty($remaining)">
				<!-- Base case: all elements sorted -->
				<xsl:sequence select="$sorted"/>
			</xsl:when>
			<xsl:otherwise>
				<!-- Get baseClasses of already sorted elements -->
				<xsl:variable name="sorted-baseClasses" select="
					for $elem in $sorted 
					return string($elem/@baseClass)
				"/>
				
				<!-- Find elements whose dependencies are all satisfied -->
				<xsl:variable name="ready-elements" select="
					for $elem in $remaining
					return
						let $elem-baseClass := string($elem/@baseClass),
							$elem-deps := $deps-map($elem-baseClass)
						return
							if (every $dep in $elem-deps satisfies $dep = $sorted-baseClasses) then
								$elem
							else
								()
				"/>
				
				<xsl:choose>
					<xsl:when test="exists($ready-elements)">
						<!-- Process ready elements and recurse -->
						<xsl:variable name="new-sorted" select="($sorted, $ready-elements)"/>
						<xsl:variable name="new-remaining" select="$remaining except $ready-elements"/>
						<xsl:sequence select="fn:topological-sort-helper($new-remaining, $deps-map, $new-sorted)"/>
					</xsl:when>
					<xsl:otherwise>
						<!-- Circular dependency detected - fall back to document order -->
						<xsl:sequence select="($sorted, $remaining)"/>
					</xsl:otherwise>
				</xsl:choose>
			</xsl:otherwise>
		</xsl:choose>
	</xsl:function>

	<!-- 
		Function: fn:get-root-fields
		Purpose: Identifies top-level Root classes that are never referenced by other classes.
				 These are the true "message roots" that should appear at the top level of the schema.
				 Handles both direct references and references through abstract classes (unions).
		Parameters: 
			$all-roots - All a:Root elements in the profile
		Returns: element()* - Sequence of Root elements that are never referenced by any other class
	-->
	<xsl:function name="fn:get-root-fields" as="element()*">
		<xsl:param name="all-roots" as="element()*"/>
		
		<xsl:message select="'=== fn:get-root-fields Debug ==='"/>
		<xsl:message select="concat('Total Roots: ', count($all-roots))"/>
		
		<!-- Build a set of all baseClass values referenced by Instance/Reference in ANY Root or ComplexType -->
		<xsl:variable name="direct-references" select="
			distinct-values(
				for $element in root($all-roots[1])//(a:Root | a:ComplexType)
				return
					for $child in $element/(a:Instance | a:Reference)
					return string($child/@baseClass)
			)
		"/>
		
		<xsl:message select="concat('Direct references count: ', count($direct-references))"/>
		<xsl:for-each select="$direct-references">
			<xsl:message select="concat('  Direct ref: ', .)"/>
		</xsl:for-each>
		
		<!-- For each direct reference, find all Root classes that inherit from it -->
		<xsl:variable name="indirect-references" select="
			distinct-values(
				for $ref in $direct-references
				return
					(: Get all Root classes that inherit from this reference :)
					for $root in $all-roots
					return
						if (fn:inherits-from($root, $ref))
						then string($root/@baseClass)
						else ()
			)
		"/>
		
		<xsl:message select="concat('Indirect references count: ', count($indirect-references))"/>
		<xsl:for-each select="$indirect-references">
			<xsl:message select="concat('  Indirect ref: ', .)"/>
		</xsl:for-each>
		
		<!-- Combine direct and indirect references -->
		<xsl:variable name="all-references" select="($direct-references, $indirect-references)"/>
		
		<xsl:message select="concat('Total references (combined): ', count(distinct-values($all-references)))"/>
		
		<!-- Return roots whose baseClass is NOT in the referenced set -->
		<xsl:variable name="result" select="
			for $root in $all-roots
			return
				if (not(string($root/@baseClass) = $all-references))
				then $root
				else ()
		"/>
		
		<xsl:message select="concat('Root fields result count: ', count($result))"/>
		<xsl:for-each select="$result">
			<xsl:message select="concat('  Root field: ', @name, ' (', @baseClass, ')')"/>
		</xsl:for-each>
		
		<xsl:sequence select="$result"/>
	</xsl:function>
		
	<xsl:template name="generate-fields">
		<xsl:choose>
			<xsl:when test="a:SuperType">
				<xsl:variable name="supertype_name" select="a:SuperType/@name"/>
				<xsl:for-each select="/*/node()[@name = $supertype_name]">
					<xsl:call-template name="generate-fields"/>
				</xsl:for-each>
				<xsl:apply-templates select="a:Complex|a:Enumerated|a:Compound|a:SimpleEnumerated|a:Simple|a:Domain|a:Instance|a:Reference|a:Choice"/>
			</xsl:when>
			<xsl:otherwise>
				<xsl:apply-templates select="a:Complex|a:Enumerated|a:Compound|a:SimpleEnumerated|a:Simple|a:Domain|a:Instance|a:Reference|a:Choice"/>
			</xsl:otherwise>
		</xsl:choose>
	</xsl:template>
				
	<xsl:template match="a:Catalog">
		<!-- the top level template -->
		<document>
			<list begin="[" indent="     " delim="," end="]">
			
				<!-- Apache Avro instance data header definition. -->
				<list begin="{{" indent="     " delim="," end="}}">
					<item>"type": "record"</item>
					<item>"name": "Header"</item>
					<item>"namespace": "<xsl:value-of select="$package_prefix"/>"</item>
					<list begin="&quot;fields&quot;: [" indent="     " delim="," end="]">
						<list begin="{{" indent="     " delim="," end="}}">
							<item>"name": "profProfile"</item>
							<item>"type": "string"</item>
							<item>"doc": "URI of the DX-PROF profile this dataset conforms to, e.g. https://ap.cim4.eu/StateVariables/3.0"</item>
						</list>
						<list begin="{{" indent="     " delim="," end="}}">
							<item>"name": "identifier"</item>
							<item>"type": "string"</item>
							<item>"doc": "Dataset identifier, aligned with dcterms:identifier / dcat:Dataset.@about."</item>
						</list>
						<list begin="{{" indent="     " delim="," end="}}">
							<item>"name": "isVersionOf"</item>
							<item>"type": [ "null", "string" ]</item>
							<item>"doc": "URI of the logical dataset or model this is a version of (dct:isVersionOf)."</item>
						</list>
						<list begin="{{" indent="     " delim="," end="}}">
							<item>"name": "version"</item>
							<item>"type": [ "null", "string" ]</item>
							<item>"doc": "Version label for this dataset instance (aligned with dcat:version). Useful when multiple messages represent different versions of the same time slice."</item>
						</list>
						<list begin="{{" indent="     " delim="," end="}}">
							<item>"name": "startDate"</item>
							<item>"type": "string"</item>
							<item>"doc": "Start of the validity interval for this dataset, aligned with dcat:startDate (ISO-8601). Typically the case time of the power system state."</item>
						</list>
						<list begin="{{" indent="     " delim="," end="}}">
							<item>"name": "schemaRef"</item>
							<item>"type": "string"</item>
							<item>"doc": "Dereferenceable URI or Schema Registry URL for the Avro schema used to encode this dataset."</item>
						</list>					
					</list>
				</list>
			
				<!-- Enumerations can be generated first as they should have no dependencies -->
				<xsl:apply-templates select="a:EnumeratedType"/>
				
				<!-- Step 1: Create exclusion map for EnumeratedTypes ONLY -->
				<xsl:variable name="compound-exclusions" select="fn:create-exclusions-map(//a:EnumeratedType)"/>
				
				<!-- Step 2: Create dependency map for CompoundTypes. Note we exclude any dependencies to enumerations since they've been processed. -->
				<xsl:variable name="compound-deps-map" select="fn:build-dependencies-map(//a:CompoundType, $compound-exclusions)"/>

				<!-- Step 3: Process CompoundTypes in correct dependency order. -->
				<xsl:variable name="sorted-compound-types" select="fn:topological-sort(//a:CompoundType, $compound-deps-map)"/>
				<xsl:apply-templates select="$sorted-compound-types"/>
				
				<!-- Step 4: Create exclusion map for EnumeratedTypes AND CompoundTypes (already processed) -->
				<xsl:variable name="root-exclusions" select="fn:create-exclusions-map(//a:EnumeratedType|//a:CompoundType)"/>
				
				<!-- Step 5: Finally, we create dependency map for Root types excluding EnumeratedTypes and CompoundTypes -->
				<!-- Note that we do not include a:ComplexType as they are abstract and Avro does not support inheritance -->
				<!-- and instead the inheritance hierarchy is "flattened" and all attributes and associations are in the  -->
				<!-- concrete (i.e. a:Root) classes.                                                                      -->
				<xsl:variable name="root-deps-map" select="fn:build-dependencies-map(//a:Root, $root-exclusions)"/>
				
				<!-- Step 6: Process Root elements in correct dependency order -->
				<xsl:variable name="sorted-root-types" select="fn:topological-sort(//a:Root, $root-deps-map)"/>
				<xsl:apply-templates select="$sorted-root-types"/>
				
				<!-- The final step is to create the 'document wrapper' derived from the name of the profile -->
				<list begin="{{" indent="     " delim="," end="}}">
					<xsl:if test="$copyright-single-line and $copyright-single-line != ''">
						<item>"copyright": "<xsl:value-of select="$copyright-single-line" disable-output-escaping="yes"/>"</item>			
					</xsl:if>
					<item>"type": "record"</item>
					<item>"name": "<xsl:value-of select="$envelope"/>"</item>
					<item>"namespace": "<xsl:value-of select="$package_prefix"/>"</item>
					<item>"doc": "<xsl:call-template name="annotate"/>"</item>
					<xsl:if test="a:Root">
						<list begin="&quot;fields&quot;: [" indent="     " delim="," end="]">
							<!-- Get all Root elements -->
							<xsl:variable name="all-roots" select="//a:Root"/>
							
							<!-- Find true root fields (never referenced) -->
							<xsl:variable name="root-fields" select="fn:get-root-fields($all-roots)"/>
						
							<!-- Process only the root fields -->
							<xsl:for-each select="$root-fields">
								<xsl:message select="concat('Root field: ', @name, ' (', @baseClass, ')')"/>
							</xsl:for-each>

							<xsl:for-each select="$root-fields">
								<list begin="{{" indent="     " delim="," end="}}">
									<item>"name": "<xsl:value-of select="@name"/>"</item>
									<xsl:choose>
										<xsl:when test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
											<list begin="&quot;type&quot;: {{" indent="    " delim="," end="}}">
												<item>"type": "array"</item>
												<item>"items": "<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
											</list>
											<xsl:if test="@minOccurs = 0">
												<item>"default": []</item>
											</xsl:if>
										</xsl:when>	
										<xsl:otherwise>
											<item>"type": "<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
										</xsl:otherwise>
									</xsl:choose>
									<item>"doc": "<xsl:call-template name="annotate"/>"</item>
									<xsl:if test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
										<xsl:if test="@minOccurs != 0">
											<item>"minCardDoc": "[min cardinality = <xsl:value-of select="@minOccurs"/>] Application level validation will be required to ensure the array contains at least <xsl:value-of select="@minOccurs"/> <xsl:choose><xsl:when test="@minOccurs = 1"> item.</xsl:when><xsl:otherwise> items.</xsl:otherwise></xsl:choose>"</item>
										</xsl:if>
										<xsl:if test="not(@maxOccurs = 'unbounded')">
											<item>"maxCardDoc": "[max cardinality = <xsl:value-of select="@maxOccurs"/>] Application level validation will be required to ensure the array contains at most <xsl:value-of select="@maxOccurs"/> items."</item>
										</xsl:if>
									</xsl:if>	
								</list>
							</xsl:for-each>
						</list>
					</xsl:if>
				</list>		
							
			</list>
		</document>
	</xsl:template>
				
	<!-- Note that for AVRO schemas we don't generate Root classes that don't have any attributes. These are considered association references to "external" entities. -->
	<xsl:template match="a:Root">
		<xsl:variable name="fieldCount" select="fn:count-fields(.)"/>
		<xsl:if test="$fieldCount > 0">
			<list begin="{{" indent="     " delim="," end="}}">
				<item>"type": "record"</item>
				<item>"name": "<xsl:value-of select="@name"/>"</item>
				<item>"namespace": "<xsl:value-of select="fn:package-name(., $package_prefix)"/>"</item>
				<item>"doc": "<xsl:call-template name="annotate"/>"</item>
				<item>"modelReference": "<xsl:value-of select="fn:model-reference(.)"/>"</item>
				<list begin="&quot;fields&quot;: [" indent="     " delim="," end="]">
					<xsl:call-template name="generate-fields"/>
				</list>
			</list>
		</xsl:if>
    </xsl:template>
    
    <xsl:template match="a:CompoundType">
		<list begin="{{" indent="     " delim="," end="}}">
			<item>"type": "record"</item>
			<item>"name": "<xsl:value-of select="@name"/>"</item>
			<item>"namespace": "<xsl:value-of select="fn:package-name(., $package_prefix)"/>"</item>
			<item>"doc": "<xsl:call-template name="annotate"/>"</item>
			<item>"modelReference": "<xsl:value-of select="fn:model-reference(.)"/>"</item>
			<list begin="&quot;fields&quot;: [" indent="     " delim="," end="]">
				<xsl:call-template name="generate-fields"/>
			</list>
		</list>
    </xsl:template>
    	
    <xsl:template match="a:EnumeratedType">
		<list begin="{{" indent="     " delim="," end="}}">
			<item>"type": "enum"</item>
			<item>"name": "<xsl:value-of select="@name"/>"</item>
			<item>"namespace": "<xsl:value-of select="fn:package-name(., $package_prefix)"/>"</item>
			<item>"doc": "<xsl:call-template name="annotate"/>"</item>
			<item>"modelReference": "<xsl:value-of select="fn:model-reference(.)"/>"</item>
			<list begin="&quot;symbols&quot;: [" indent="     " delim="," end="]">
				<xsl:for-each select="a:EnumeratedValue">
					<item>"<xsl:value-of select="@name"/>"</item>
				</xsl:for-each>
			</list>
		</list>
	</xsl:template>
	
	<xsl:template match="a:Simple|a:Domain">
		<list begin="{{" indent="     " delim="," end="}}">
			<xsl:variable name="type" select="fn:get-avro-type(@xstype)"/>
			<item>"name": "<xsl:value-of select="@name"/>"</item>
			<xsl:choose>
				<xsl:when test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
					<list begin="&quot;type&quot;: {{" indent="    " delim="," end="}}">
						<item>"type": "array"</item>
						<item>"items": <xsl:value-of select="$type"/></item>
					</list>
					<xsl:if test="@minOccurs = 0">
						<item>"default": []</item>
					</xsl:if>
				</xsl:when>	
				<xsl:otherwise>
					<xsl:choose>
						<xsl:when test="@minOccurs = 0">
							<item>"type": [ "null", <xsl:value-of select="$type"/> ]</item>
							<item>"default": null</item>
						</xsl:when>
						<xsl:otherwise>
							<item>"type": <xsl:value-of select="$type"/></item>
						</xsl:otherwise>
					</xsl:choose>
				</xsl:otherwise>
			</xsl:choose>
			<item>"doc": "<xsl:call-template name="annotate"/>"</item>
			<item>"modelReference": "<xsl:value-of select="fn:model-reference(.)"/>"</item>
			<xsl:if test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
				<xsl:if test="@minOccurs != 0">
					<item>"minCardDoc": "[min cardinality = <xsl:value-of select="@minOccurs"/>] Application level validation will be required to ensure the array contains at least <xsl:value-of select="@minOccurs"/> <xsl:choose><xsl:when test="@minOccurs = 1"> item.</xsl:when><xsl:otherwise> items.</xsl:otherwise></xsl:choose>"</item>
				</xsl:if>
				<xsl:if test="not(@maxOccurs = 'unbounded')">
					<item>"maxCardDoc": "[max cardinality = <xsl:value-of select="@maxOccurs"/>] Application level validation will be required to ensure the array contains at most <xsl:value-of select="@maxOccurs"/> items."</item>
				</xsl:if>
			</xsl:if>	
		</list>
	</xsl:template>
	
	<xsl:template match="a:Compound|a:Enumerated">
		<list begin="{{" indent="     " delim="," end="}}">
			<item>"name": "<xsl:value-of select="@name"/>"</item>
			<xsl:choose>
				<xsl:when test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
					<list begin="&quot;type&quot;: {{" indent="    " delim="," end="}}">
						<item>"type": "array"</item>
						<item>"items": "<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
					</list>
					<xsl:if test="@minOccurs = 0">
						<item>"default": []</item>
					</xsl:if>
				</xsl:when>	
				<xsl:otherwise>
					<xsl:choose>
						<xsl:when test="@minOccurs = 0">
							<list begin="&quot;type&quot;: [" indent="     " delim="," end="]">
								<item>"null"</item>
								<item>"<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
							</list>
							<item>"default": null</item>
						</xsl:when>
						<xsl:otherwise>
							<item>"type": "<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
						</xsl:otherwise>
					</xsl:choose>
				</xsl:otherwise>
			</xsl:choose>
			<item>"doc": "<xsl:call-template name="annotate"/>"</item>
			<item>"modelReference": "<xsl:value-of select="fn:model-reference(.)"/>"</item>
			<xsl:if test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
				<xsl:if test="@minOccurs != 0">
					<item>"minCardDoc": "[min cardinality = <xsl:value-of select="@minOccurs"/>] Application level validation will be required to ensure the array contains at least <xsl:value-of select="@minOccurs"/> <xsl:choose><xsl:when test="@minOccurs = 1"> item.</xsl:when><xsl:otherwise> items.</xsl:otherwise></xsl:choose>"</item>
				</xsl:if>
				<xsl:if test="not(@maxOccurs = 'unbounded')">
					<item>"maxCardDoc": "[max cardinality = <xsl:value-of select="@maxOccurs"/>] Application level validation will be required to ensure the array contains at most <xsl:value-of select="@maxOccurs"/> items."</item>
				</xsl:if>
			</xsl:if>	
		</list>
	</xsl:template>
	
	<xsl:template match="a:Instance|a:Reference">
		<xsl:variable name="baseClass" select="@baseClass"/>
		<xsl:variable name="theClass" select="//a:Root[@baseClass = $baseClass]|//a:ComplexType[@baseClass = $baseClass]"/>
		<xsl:variable name="fullyQualifiedClass" select="fn:fully-qualified-class($theClass, $package_prefix)"/>
		<xsl:variable name="fieldCount" select="fn:count-fields($theClass)"/>
		
		<!-- Check if this references an abstract class (ComplexType) that has concrete subclasses -->
		<xsl:variable name="isAbstractWithSubclasses">
			<xsl:choose>
				<xsl:when test="//a:ComplexType[@baseClass = $baseClass]">
					<!-- This references a ComplexType - check if there are Root subclasses -->
					<xsl:variable name="concreteSubclasses">
						<xsl:for-each select="//a:Root">
							<xsl:if test="fn:inherits-from(., $baseClass)">
								<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>
							</xsl:if>
						</xsl:for-each>
					</xsl:variable>
					<xsl:choose>
						<xsl:when test="string-length($concreteSubclasses) > 0">
							<xsl:text>true</xsl:text>
						</xsl:when>
						<xsl:otherwise>
							<xsl:text>false</xsl:text>
						</xsl:otherwise>
					</xsl:choose>
				</xsl:when>
				<xsl:otherwise>
					<xsl:text>false</xsl:text>
				</xsl:otherwise>
			</xsl:choose>
		</xsl:variable>
		
		<list begin="{{" indent="     " delim="," end="}}">	
			<xsl:choose>
				<xsl:when test="$fieldCount > 0">
					<item>"name": "<xsl:value-of select="@name"/>"</item>
					<xsl:choose>
						<xsl:when test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
							<list begin="&quot;type&quot;: {{" indent="    " delim="," end="}}">
								<item>"type": "array"</item>
								<xsl:choose>
									<!-- Union case: array of union types -->
									<xsl:when test="$isAbstractWithSubclasses = 'true'">
										<list begin="&quot;items&quot;: [" indent="     " delim="," end="]">
											<xsl:for-each select="//a:Root">
												<xsl:if test="fn:inherits-from(., $baseClass)">
													<item>"<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
												</xsl:if>
											</xsl:for-each>
										</list>
									</xsl:when>
									<!-- Normal case: array of single type -->
									<xsl:otherwise>
										<item>"items": "<xsl:value-of select="$fullyQualifiedClass"/>"</item>
									</xsl:otherwise>
								</xsl:choose>
							</list>
							<xsl:if test="@minOccurs = 0">
								<item>"default": []</item>
							</xsl:if>
						</xsl:when>	
						<xsl:otherwise>
							<xsl:choose>
								<xsl:when test="@minOccurs = 0">
									<xsl:choose>
										<!-- Union case: nullable union of types -->
										<xsl:when test="$isAbstractWithSubclasses = 'true'">
											<list begin="&quot;type&quot;: [" indent="     " delim="," end="]">
												<xsl:for-each select="//a:Root">
													<xsl:if test="fn:inherits-from(., $baseClass)">
														<item>"<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
													</xsl:if>
												</xsl:for-each>
											</list>
										</xsl:when>
										<!-- Normal case: nullable single type -->
										<xsl:otherwise>
											<list begin="&quot;type&quot;: [" indent="     " delim="," end="]">
												<item>"null", "<xsl:value-of select="$fullyQualifiedClass"/>"</item>
											</list>
										</xsl:otherwise>
									</xsl:choose>
									<item>"default": null</item>
								</xsl:when>
								<xsl:otherwise>
									<xsl:choose>
										<!-- Union case: required union of types -->
										<xsl:when test="$isAbstractWithSubclasses = 'true'">
											<list begin="&quot;type&quot;: [" indent="     " delim="," end="]">
												<xsl:for-each select="//a:Root">
													<xsl:if test="fn:inherits-from(., $baseClass)">
														<item>"<xsl:value-of select="fn:fully-qualified-class(., $package_prefix)"/>"</item>
													</xsl:if>
												</xsl:for-each>
											</list>
										</xsl:when>
										<!-- Normal case: required single type -->
										<xsl:otherwise>
											<item>"type": "<xsl:value-of select="$fullyQualifiedClass"/>"</item>
										</xsl:otherwise>
									</xsl:choose>
								</xsl:otherwise>
							</xsl:choose>
						</xsl:otherwise>
					</xsl:choose>
					<item>"doc": "<xsl:call-template name="annotate"/>"</item>
					<item>"modelReference": "<xsl:value-of select="fn:model-reference(.)"/>"</item>
					<xsl:if test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
						<xsl:if test="@minOccurs != 0">
							<item>"minCardDoc": "[min cardinality = <xsl:value-of select="@minOccurs"/>] Application level validation will be required to ensure the array contains at least <xsl:value-of select="@minOccurs"/> <xsl:choose><xsl:when test="@minOccurs = 1"> item.</xsl:when><xsl:otherwise> items.</xsl:otherwise></xsl:choose>"</item>
						</xsl:if>
						<xsl:if test="not(@maxOccurs = 'unbounded')">
							<item>"maxCardDoc": "[max cardinality = <xsl:value-of select="@maxOccurs"/>] Application level validation will be required to ensure the array contains at most <xsl:value-of select="@maxOccurs"/> items."</item>
						</xsl:if>
					</xsl:if>
				</xsl:when>
				<xsl:otherwise>
					<item>"name": "<xsl:value-of select="@name"/>"</item>
					<xsl:choose>
						<xsl:when test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
							<list begin="&quot;type&quot;: {{" indent="    " delim="," end="}}">
								<item>"type": "array"</item>
								<item>"items": "string"</item>
							</list>
							<xsl:if test="@minOccurs = 0">
								<item>"default": []</item>
							</xsl:if>
						</xsl:when>	
						<xsl:otherwise>
							<xsl:choose>
								<xsl:when test="@minOccurs = 0">
									<item>"type": [ "null", "string" ]</item>
									<item>"default": null</item>
								</xsl:when>
								<xsl:otherwise>
									<item>"type": "string"</item>
								</xsl:otherwise>
							</xsl:choose>
						</xsl:otherwise>
					</xsl:choose>
					<item>"doc": "<xsl:call-template name="annotate"/> Note that the value of this field is the identifier (e.g. mRID) used to reference the <xsl:value-of select="@type"/> external to this profile."</item>
					<item>"modelReference": "<xsl:value-of select="fn:model-reference(.)"/>"</item>
					<xsl:if test="@maxOccurs = 'unbounded' or @maxOccurs &gt; 1">
						<xsl:if test="@minOccurs != 0">
							<item>"minCardDoc": "[min cardinality = <xsl:value-of select="@minOccurs"/>] Application level validation will be required to ensure the array contains at least <xsl:value-of select="@minOccurs"/> <xsl:choose><xsl:when test="@minOccurs = 1"> item.</xsl:when><xsl:otherwise> items.</xsl:otherwise></xsl:choose>"</item>
						</xsl:if>
						<xsl:if test="not(@maxOccurs = 'unbounded')">
							<item>"maxCardDoc": "[max cardinality = <xsl:value-of select="@maxOccurs"/>] Application level validation will be required to ensure the array contains at most <xsl:value-of select="@maxOccurs"/> items."</item>
						</xsl:if>
					</xsl:if>
				</xsl:otherwise>
			</xsl:choose>
		</list>
	</xsl:template>
	
	<xsl:template name="annotate">
		<xsl:apply-templates mode="annotate"/>
	</xsl:template>
	
	<xsl:template match="a:Comment|a:Note" mode="annotate">
		<!-- Remove double quotes to eliminate broken comments, etc. -->
		<xsl:value-of select="translate(., '&quot;', '')"/>
	</xsl:template>
	
	<xsl:template match="text()">
		<!-- dont pass text through -->
	</xsl:template>
	
	<xsl:template match="node()" mode="annotate">
		<!-- dont pass any defaults in annotate mode -->
	</xsl:template>
	
	<xsl:template match="node()" mode="declare">
		<!-- dont pass any defaults in declare mode -->
	</xsl:template>
	
</xsl:stylesheet>
```
