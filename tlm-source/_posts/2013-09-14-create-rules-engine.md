---
layout: post
title: "Create Your Own Rules Engine Using MVEL and XML"
tagline: "Blueprint for XpressIt, a full featured Rules Engine"
description: "Using this post I will show you how to create your own rules engine using MVEL(an expression language) and XML"
category: articles
tags: [java rules engine, rules engine, mvel, xml]
authors: [goashok,smtechnocrat]
image:
  feature: texture-feature-04.jpg
  credit: Texture Lovers
  creditlink: http://texturelovers.com
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

MVEL is a powerful expression language for Java-based applications. It provides a plethora of features and is suited for everything from the smallest property binding and extraction, to full blown scripts.
Rules are defined as expressions which are evaluated at runtime on the facts provided. So that explains our choice of MVEL. Now to define the rules and to have a persistent store for our rules, we choose
XML as most people have worked with XML in some shape or form. Using XML to define our rules also obviates the need of a special editor as most IDEs provide built-in support for XML. Now using these two key
tools lets start building our rules engine. Note that a full featured open source rules engine (XpressIt) is available for you using the same blue print that we will be discussing in this article. 

## Rule Definition In XML
{% highlight xml %}
<rule name="defaultTrader">
	<condition>
		swap.traderId == null;
	</condition>
	<action>
		swap.traderId = 'system';
	</action>
</rule>	
{% endhighlight %}
In this rule, `swap.traderId == null` is a mvel expression. The expression implies that at runtime , the context will have a variable called swap referring to an object which will have a field called traderId.
Note, that condition is always a boolean expression returning true or false. If the expression evaluates to true, we would like to execute an action which is defined in the rule under `<action>` element. Note that
`swap.traderId = 'system'` is an assignment expression in mvel. The rule essentially checks to see if the traderId is null, then assign traderId to 'system'.

## Code To Parse XML And Evaluate Rule
{% highlight java %}
//key imports
import org.mvel2.MVEL;
import org.mvel2.ParserContext;
import org.mvel2.compiler.CompiledExpression;
import org.mvel2.compiler.ExpressionCompiler;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

..........
.....
......

SAXReader reader = new SAXReader();
Document document = reader.read(ruleFileDir);
Element root = document.getRootElement();
Element ruleElement = root.elements("rule").get(0);
String ruleName = ruleElement.attributeValue("name");
String condition = ruleElement.elements("condition").get(0).getText();
String action = ruleElement.elements("action").get(0).getText();
Expression conditionExpression = new ExpressionCompiler(condition).compile(new ParserContext());
Expression actionExpression = new ExpressionCompiler(action).compile(new ParserContext());
//Assuming we have a class called Swap with field traderId and few other fields
//Lets construct a new Swap with no fields initialized.
Swap s = new Swap();
Map context = new HashMap();
context.put("swap", s);
boolean result = (Boolean) MVEL.executeExpression(conditionExpression, context);
if(result == true) //if the condition was true, we now evaluate the actionExpression to set the traderId.
{
	MVEL.executeExpression(actionExpression, context);
}
{% endhighlight %}

In the sections above, we just covered the basics of rule evaluation. But in the full featured implementation, we have to do a little more than that.
For example, we cannot compile the expression each time a rule is invoked. We have to organize the rules into rule sets. These are just a few of things we need
to consider for the full featured implementation. Lets talks about these and more in the next section.

## Blue Print 

### Features

* Ability to define Rule(s) in easy, yet extremely powerful MVEL expression language
* Categorize Rule(s) into RuleSet(s) for a logical grouping of rules
* Store Rule(s) and RuleSet(s) into a RuleBase
* Shared Rules that can be used across all rule sets in the rules engine
* Ability to retract Fact(s) during rule invocation  to make further rules in-eligible from firing
* Ability to inject Fact(s) during rule invocation to make further rules eligible for firing
* Define Global variables that can be used implicitly within any rule in rules engine
* Define RuleSet variables that can be used implicitly within any rule in a given RuleSet
* Ability to define error code and error messages within rule invocation
* Ability to add additional data to be returned within rule invocation besides the original Facts
* Handle to modified facts, errors and additional data on completion of rule firing
* Simple and elegant API
* Support for Decision Tables

### Key Classes
* <a href="https://github.com/goashok/XpressIt/blob/master/src/main/java/org/drexten/rules/engine/Fact.java">Fact</a>, representing the user data on which rules need to be evaluated
* <a href="https://github.com/goashok/XpressIt/blob/master/src/main/java/org/drexten/rules/engine/xpressit/Rule.java">Rule</a>, class representing the rule. Contains the expression that need to be evaluated
* <a href="https://github.com/goashok/XpressIt/blob/master/src/main/java/org/drexten/rules/engine/xpressit/RuleSet.java">RuleSet</a>, a collection of Rule(s) where the name of the RuleSet represents the category
* <a href="https://github.com/goashok/XpressIt/blob/master/src/main/java/org/drexten/rules/engine/xpressit/RuleBase.java">RuleBase</a>, an in-memory datastructure to store Rule(s) and RuleSet(s)
* <a href="https://github.com/goashok/XpressIt/blob/master/src/main/java/org/drexten/rules/engine/xpressit/RuleParser.java">RuleParser</a>, an XML parser that parses the rules from the xml rule definitions and creates Rule/RuleSet/RuleBase
* <a href="https://github.com/goashok/XpressIt/blob/master/src/main/java/org/drexten/rules/engine/xpressit/Expression.java">Expression</a>, an abstract class that contains the expression , designed for extension to support multiple expression languages
* <a href="https://github.com/goashok/XpressIt/blob/master/src/main/java/org/drexten/rules/engine/xpressit/MvelExpression.java">MVELExpression</a> , an extension of Expression that knows how to evaluate expressions(conditions/actions) written in MVEL language

### Rule Engine Api
{% highlight java %}
package org.drexten.rules.engine;


/**
 * 
 * copyrights Drexten Corp 2010 (Drexten SimpleApi Certified)
 * 
 * RuleEngine defines an interface for invoking rules on the provided facts.
 * A Fact is an Object which needs to be evaluated upon rule invocation.
 * These rules can be used for validation, derivation, logic flow etc.
 * The RuleEngine also allows users to categorize rules under rule sets.
 * Categorizing rules under rule sets provides better management of rules and
 * easier understanding of overall application behavior.
 * 
 * The rule engine supports user-defined variables that can be used implicitly in
 * the rules. This may be required if your rules need to access services which are
 * not passed as facts. These variables can be of global nature, meaning they will be 
 * automatically available for use in any rule. Or they can be limited in scope to a
 * ruleset, meaning they will be available to any rule with the given ruleset for which
 * the variables are defined. 
 * 
 * Current Implementations:
 *
 * XpressIt : An expression based rules engine that supports defining rules in widely
 * used java expression languages MVEL and OGNL.
 * 
 * @author ashok shamnani
 *
 */
public interface RuleEngine {
    
   /**
    * Fires the rule with the given name . The invoked rule evaluates the passed facts.
    * @param ruleName name of the rule
    * @param facts to be evaluated
    * @return <code>RuleEvalResult</code> results of rule evaluation
    * @throws RuleException
    */
    public RuleEvalResult fireRule(String ruleName, Fact... facts) throws RuleException ;    
    
    /**
     * Fires the rule with the given name in the given ruleset . The invoked rule evaluates the passed facts.
     * @param ruleName name of the rule
     * @param ruleSet name of the rule set
     * @param facts to be evaluated
     * @return <code>RuleEvalResult</code> results of rule evaluation
     * @throws RuleException
     */
    public RuleEvalResult fireRule(String ruleName, String ruleSet, Fact... facts) throws RuleException;
    
    /**
     * Fires all the rules in the given rule set. The invoked rules in rule set evaluates the passed facts.
     * If any of the rule in the rule set retracts any of the fact(s), subsequent rules in the rule set
     * relying on the availability of the fact will not be fired.
     * @param ruleSet name of the rule set
     * @param facts to be evaluated
     * @return <code>RuleEvalResult</code> results of rule evaluation
     * @throws RuleException
     */
    public RuleEvalResult fireRuleSet(String ruleSet, Fact... facts) throws RuleException;
    
    /**
     * Fires all the rules available in the rule engine. The invoked rules in rule set evaluates the passed facts.
     * If any of the rule in the rule set retracts any of the fact(s), subsequent rules in the rule set
     * relying on the availability of the fact will not be fired.
     * @param ruleSet name of the rule set
     * @param facts to be evaluated
     * @return <code>RuleEvalResult</code> results of rule evaluation
     * @throws RuleException
     */
    public RuleEvalResult fireAllRules(Fact... facts) throws RuleException;
    
}


{% endhighlight %}

## XpressIt

XpressIt is a full featured implementation of rules engine based in the blueprint discussed above. Find below the download links and QuickStart guide for
XpressIt

* <a href="https://github.com/goashok/XpressIt">XpressIt</a>, You can find the test classes in src/main/test.
* <a href="http://www.drexten.com/documentation/?page_id=27">QuickStart</a>, A quick start guide that covers all the features supported by XpressIt. Highly recommend
to take a look in case you need to build your own implementation.