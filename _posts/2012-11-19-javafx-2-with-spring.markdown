---
layout: post
title: JavaFX 2 with Spring
date: '2012-11-19T17:40:00.003+01:00'
author: Koen Serneels
img: 2.png
tags:
- Java SE
modified_time: '2012-11-19T20:37:24.965+01:00'
thumbnail: http://2.bp.blogspot.com/-E7kb_CDIY8U/UKpbtCztK5I/AAAAAAAAAQ0/9xGeHW-THNo/s72-c/market_place.png
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-3354319054800645126
blogger_orig_url: http://koenserneels.blogspot.com/2012/11/javafx-2-with-spring.html
---

I'm going to start this one with a bold statement: I always liked Java Swing, or applets for that matter. There, I said it.  If I perform some self analysis, this admiration probably started when I got introduced to Java.  Swing was (practically) the first thing I ever did with Java that gave some statisfactionary result and made me able to do something with the language at the time.  When I was younger we build home-brew fat clients to manage our 3.5" floppy/CD collection (written in VB and before that in basic) this probably also played a role. 

Anyway, enough about my personal rareness. Fact is that Swing has helped many build great applications but as we all know Swing has it drawbacks.   For starters it hasn't been evolving since well, a long time. It also requires a lot of boiler plate code if you want to create high quality code. It comes shipped with some quirky design "flaws", lacks out of the box patterns such as MVC. Styling is a bit of a limitation since you have to fall back on the limited L&F architecture, I18N is not build in by default and so on.  One could say that developing Swing these days is well, basically going back into time. 

Fortunately Oracle tried to change this some years ago by Launching JavaFX.  I recall getting introduced to JavaFX on Devoxx (or Javapolis as it was named back then).   The nifty demo's looked very promising, so I was glad to see that a Swing successor was finally on its way.  This changed from the moment I saw its internals. One of its major drawbacks was that it was based on a dark new syntax (called JavaFX script).  In case you have never seen JavaFX script; it looks like a bizarre breed between Java, JSON and JavaScript.  Although it is compiled to Java byte-code, and you could use the Java API's from it, integration with Java was never really good. 

The language itself (although pretty powerful) required you to spend a lot of time understanding the details, for ending up with, well, again source code, but this time less manageable and supported then plain Java code.  As it turned out, I wasn't the only one. A lot of people felt the same (for sure there were other reasons as well) and JavaFX never was a great success. 

However, a while ago Oracle changed the tide by introducing JavaFX 2.  <br>First of all they got rid of JavaFX script (which is no longer supported) and turned it into a real native Java SE API (JavaFX 2.2.3 is part of the Java 7 SE update 6) . The JavaFX API now looks more like the familiar Swing API, which is a good thing.  It gives you layout managers lookalikes, event listeners, and all those other components you were so used to, but even better. So if you want you can code JavaFX like you did Swing you can, albeit with slightly different syntax and improved architecture.  It is also possible now to <a target="_blank" href="http://docs.oracle.com/javafx/2/swing/overview.htm#CJAHBAHA">intermix existing Java Swing applications with JavaFX</a>.

But there is more. They introduced an XML based markup language that allows you to describe the view. This has some advantages, first of all coding in XML works faster then Java.  XML can be more easily be generated then Java and the syntax for describing a view is simply more compact. It is also more intuitive to express a view using some kind of markup, especially if you ever did some web development before.  So, one can have the view described in FXML (thats how its called), the application controllers separate from the view, both in Java, and your styling in CSS (yeah, so no more L&F, CSS support is standard).  You can still embed Java (or other languages) directly in the FXML; but this is probably not what you want (scriptlet anti-pattern).  Another nice thing is support for binding. You can bind each component in your view to the application controller by putting an fx:id attribute on the view component and an @FXML annotation on the instance variable in the application controller.  The corresponding element will then be auto injected, so you can change its data or behavior from inside your application controller.  It also turns out that with some lines of code you can painlessly integrate the DI framework of your choice, isn't that sweet? 

And what about the tooling? 
Well, first of all there is a plug-in for Eclipse (fxclipse) which will render you FXML on the fly. You can install it via Eclipse market place:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/market_place-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/market_place-small.png"/></a></div>

The plug-in will render any adjustment you make immediately:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/fxclipse-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/fxclipse-small.png"/></a></div>

Note that you need at least JDK7u6 for this plug-in to work. If your JDK is too old you'll get an empty pane in eclipse. Also, if you create a JavaFX project I needed to put the jfxrt.jar manually on my build classpath. You'll find this file in %JAVA_HOME%/jre/lib. 

Up until know the plug-in doesn't help you visually (by drag& drop) but that there a separate IDE: <a href="http://www.oracle.com/technetwork/java/javafx/tools/index.html" target="_blank">scene builder</a>. This builder is also integrated in Netbeans, for AFAIK there is no support for eclipse yet so you'll have to run it separately if you want to use it. The builder lets you develop FXML the visual way, using drag&drop. Nice detail; scene builder is in fact written in JavaFX. Then you also have a separate application called <a target="_blank" href="http://fxexperience.com/scenic-view/">scenic view</a> which does introspection on a running JavaFX application and shows how it is build up. You get a graph with the different nodes and their hierarchical structure. For each node you can see its properties and so forth: 

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scenic_view-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scenic_view.png"/></a></div>

Ok, so lets start with some code examples. The first thing I did was design my demo application in scene builder:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scene_builder_1-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scene_builder_1-small.png"/></a></div>

I did this graphically by d&d the containers/controlers on to the view. I also gave the controls that I want to bind to my view and fx:id, you can do that also via scene builder:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scene_builder_2-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scene_builder_2-small.png"/></a></div>
For the buttons in particular I also added an onAction (which is the method that should be executed on the controller once the button is clicked):

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scene_builder_3-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/scene_builder_3-small.png"/></a></div>

Next I added the controller manually in the source view in eclipse. There can only be one controller per FXML and it should be declared in the top level element. I made two FXML's, one that represents the main screen and one that acts as the menu bar. You probably want a division of your logic in multiple controllers, rather then stuffing to much in a single controller – single responsibility is a good design guideline here. The first FXML is “search.fxml” and represents the search criteria and result view:

<pre class="brush: xml; highlight: [11, 17]">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;

&lt;?import java.lang.*?&gt;
&lt;?import java.util.*?&gt;
&lt;?import javafx.scene.control.*?&gt;
&lt;?import javafx.scene.control.Label?&gt;
&lt;?import javafx.scene.control.cell.*?&gt;
&lt;?import javafx.scene.layout.*?&gt;
&lt;?import javafx.scene.paint.*?&gt;

&lt;StackPane id=&quot;StackPane&quot; maxHeight=&quot;-Infinity&quot; maxWidth=&quot;-Infinity&quot; minHeight=&quot;-Infinity&quot; minWidth=&quot;-Infinity&quot; prefHeight=&quot;400.0&quot; prefWidth=&quot;600.0&quot; xmlns:fx=&quot;http://javafx.com/fxml&quot; fx:controller=&quot;be.error.javafx.controller.SearchController&quot;&gt;
  &lt;children&gt;
    &lt;SplitPane dividerPositions=&quot;0.39195979899497485&quot; focusTraversable=&quot;true&quot; orientation=&quot;VERTICAL&quot; prefHeight=&quot;200.0&quot; prefWidth=&quot;160.0&quot;&gt;
      &lt;items&gt;
        &lt;GridPane fx:id=&quot;grid&quot; prefHeight=&quot;91.0&quot; prefWidth=&quot;598.0&quot;&gt;
          &lt;children&gt;
     &lt;fx:include source=&quot;/menu.fxml&quot;/&gt;
            &lt;GridPane prefHeight=&quot;47.0&quot; prefWidth=&quot;486.0&quot; GridPane.columnIndex=&quot;1&quot; GridPane.rowIndex=&quot;5&quot;&gt;
              &lt;children&gt;
                &lt;Button fx:id=&quot;clear&quot; cancelButton=&quot;true&quot; mnemonicParsing=&quot;false&quot; onAction=&quot;#clear&quot; text=&quot;Clear&quot; GridPane.columnIndex=&quot;1&quot; GridPane.rowIndex=&quot;1&quot; /&gt;
                &lt;Button fx:id=&quot;search&quot; defaultButton=&quot;true&quot; mnemonicParsing=&quot;false&quot; onAction=&quot;#search&quot; text=&quot;Search&quot; GridPane.columnIndex=&quot;2&quot; GridPane.rowIndex=&quot;1&quot; /&gt;
              &lt;/children&gt;
              &lt;columnConstraints&gt;
                &lt;ColumnConstraints hgrow=&quot;SOMETIMES&quot; maxWidth=&quot;338.0&quot; minWidth=&quot;10.0&quot; prefWidth=&quot;338.0&quot; /&gt;
                &lt;ColumnConstraints hgrow=&quot;SOMETIMES&quot; maxWidth=&quot;175.0&quot; minWidth=&quot;0.0&quot; prefWidth=&quot;67.0&quot; /&gt;
                &lt;ColumnConstraints hgrow=&quot;SOMETIMES&quot; maxWidth=&quot;175.0&quot; minWidth=&quot;10.0&quot; prefWidth=&quot;81.0&quot; /&gt;
              &lt;/columnConstraints&gt;
              &lt;rowConstraints&gt;
                &lt;RowConstraints maxHeight=&quot;110.0&quot; minHeight=&quot;10.0&quot; prefHeight=&quot;10.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
                &lt;RowConstraints maxHeight=&quot;72.0&quot; minHeight=&quot;10.0&quot; prefHeight=&quot;40.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
              &lt;/rowConstraints&gt;
            &lt;/GridPane&gt;
            &lt;Label alignment=&quot;CENTER_RIGHT&quot; prefHeight=&quot;21.0&quot; prefWidth=&quot;101.0&quot; text=&quot;Product name:&quot; GridPane.columnIndex=&quot;0&quot; GridPane.rowIndex=&quot;1&quot; /&gt;
            &lt;TextField fx:id=&quot;productName&quot; prefWidth=&quot;200.0&quot; GridPane.columnIndex=&quot;1&quot; GridPane.rowIndex=&quot;1&quot; /&gt;
            &lt;Label alignment=&quot;CENTER_RIGHT&quot; prefWidth=&quot;101.0&quot; text=&quot;Min price:&quot; GridPane.columnIndex=&quot;0&quot; GridPane.rowIndex=&quot;2&quot; /&gt;
            &lt;Label alignment=&quot;CENTER_RIGHT&quot; prefWidth=&quot;101.0&quot; text=&quot;Max price:&quot; GridPane.columnIndex=&quot;0&quot; GridPane.rowIndex=&quot;3&quot; /&gt;
            &lt;TextField fx:id=&quot;minPrice&quot; prefWidth=&quot;200.0&quot; GridPane.columnIndex=&quot;1&quot; GridPane.rowIndex=&quot;2&quot; /&gt;
            &lt;TextField fx:id=&quot;maxPrice&quot; prefWidth=&quot;200.0&quot; GridPane.columnIndex=&quot;1&quot; GridPane.rowIndex=&quot;3&quot; /&gt;
          &lt;/children&gt;
          &lt;columnConstraints&gt;
            &lt;ColumnConstraints hgrow=&quot;SOMETIMES&quot; maxWidth=&quot;246.0&quot; minWidth=&quot;10.0&quot; prefWidth=&quot;116.0&quot; /&gt;
            &lt;ColumnConstraints fillWidth=&quot;false&quot; hgrow=&quot;SOMETIMES&quot; maxWidth=&quot;537.0&quot; minWidth=&quot;10.0&quot; prefWidth=&quot;482.0&quot; /&gt;
          &lt;/columnConstraints&gt;
          &lt;rowConstraints&gt;
            &lt;RowConstraints maxHeight=&quot;64.0&quot; minHeight=&quot;10.0&quot; prefHeight=&quot;44.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
            &lt;RowConstraints maxHeight=&quot;68.0&quot; minHeight=&quot;0.0&quot; prefHeight=&quot;22.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
            &lt;RowConstraints maxHeight=&quot;68.0&quot; minHeight=&quot;10.0&quot; prefHeight=&quot;22.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
            &lt;RowConstraints maxHeight=&quot;68.0&quot; minHeight=&quot;10.0&quot; prefHeight=&quot;22.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
            &lt;RowConstraints maxHeight=&quot;167.0&quot; minHeight=&quot;10.0&quot; prefHeight=&quot;14.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
            &lt;RowConstraints maxHeight=&quot;167.0&quot; minHeight=&quot;10.0&quot; prefHeight=&quot;38.0&quot; vgrow=&quot;SOMETIMES&quot; /&gt;
          &lt;/rowConstraints&gt;
        &lt;/GridPane&gt;
        &lt;StackPane prefHeight=&quot;196.0&quot; prefWidth=&quot;598.0&quot;&gt;
          &lt;children&gt;
            &lt;TableView fx:id=&quot;table&quot; prefHeight=&quot;200.0&quot; prefWidth=&quot;200.0&quot;&gt;
              &lt;columns&gt;
                &lt;TableColumn prefWidth=&quot;120.0&quot; resizable=&quot;true&quot; text=&quot;OrderId&quot;&gt;
                  &lt;cellValueFactory&gt;
                    &lt;PropertyValueFactory property=&quot;orderId&quot; /&gt;
                  &lt;/cellValueFactory&gt;
                &lt;/TableColumn&gt;
                &lt;TableColumn prefWidth=&quot;120.0&quot; text=&quot;CustomerId&quot;&gt;
                  &lt;cellValueFactory&gt;
                    &lt;PropertyValueFactory property=&quot;customerId&quot; /&gt;
                  &lt;/cellValueFactory&gt;
                &lt;/TableColumn&gt;
                &lt;TableColumn prefWidth=&quot;120.0&quot; text=&quot;#products&quot;&gt;
                  &lt;cellValueFactory&gt;
                    &lt;PropertyValueFactory property=&quot;productsCount&quot; /&gt;
                  &lt;/cellValueFactory&gt;
                &lt;/TableColumn&gt;
                &lt;TableColumn prefWidth=&quot;120.0&quot; text=&quot;Delivered&quot;&gt;
                  &lt;cellValueFactory&gt;
                    &lt;PropertyValueFactory property=&quot;delivered&quot; /&gt;
                  &lt;/cellValueFactory&gt;
                &lt;/TableColumn&gt;
                &lt;TableColumn prefWidth=&quot;120.0&quot; text=&quot;Delivery days&quot;&gt;
                  &lt;cellValueFactory&gt;
                    &lt;PropertyValueFactory property=&quot;deliveryDays&quot; /&gt;
                  &lt;/cellValueFactory&gt;
                &lt;/TableColumn&gt;
                &lt;TableColumn prefWidth=&quot;150.0&quot; text=&quot;Total order price&quot;&gt;
                  &lt;cellValueFactory&gt;
                    &lt;PropertyValueFactory property=&quot;totalOrderPrice&quot; /&gt;
                  &lt;/cellValueFactory&gt;
                &lt;/TableColumn&gt;
              &lt;/columns&gt;
            &lt;/TableView&gt;
          &lt;/children&gt;
        &lt;/StackPane&gt;
      &lt;/items&gt;
    &lt;/SplitPane&gt;
  &lt;/children&gt;
&lt;/StackPane&gt;
</pre>On line 11 you can see that I configured the application controller class that should be used with the view. On line 17 you can see the import of the separate menu.fxml which is shown here:  

<pre class="brush: xml; highlight: [7]">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;

&lt;?import javafx.scene.control.*?&gt;
&lt;?import javafx.scene.layout.*?&gt;
&lt;?import javafx.scene.control.MenuItem?&gt;

&lt;Pane prefHeight=&quot;465.0&quot; prefWidth=&quot;660.0&quot; xmlns:fx=&quot;http://javafx.com/fxml&quot; fx:controller=&quot;be.error.javafx.controller.FileMenuController&quot;&gt;
 &lt;children&gt;
  &lt;MenuBar layoutX=&quot;0.0&quot; layoutY=&quot;0.0&quot;&gt;
   &lt;menus&gt;
    &lt;Menu mnemonicParsing=&quot;false&quot; text=&quot;File&quot;&gt;
     &lt;items&gt;
      &lt;MenuItem text=&quot;Exit&quot; onAction=&quot;#exit&quot; /&gt; 
     &lt;/items&gt;
    &lt;/Menu&gt;
   &lt;/menus&gt;
  &lt;/MenuBar&gt;
 &lt;/children&gt;
&lt;/Pane&gt;
</pre>

On line 7 you can see that it uses a different controller.  In Eclipse, if you open the fxclipse view from the plug-in you will get the same rendered view as in scene builder. Its however convenient if you want to make small changes in the code to see them directly reflected:  The code for launching the application is pretty standard:

<pre class="brush: java;">
package be.error.javafx;

import javafx.application.Application;
import javafx.scene.Parent;
import javafx.scene.Scene;
import javafx.stage.Stage;

public class TestApplication extends Application {

 private static final SpringFxmlLoader loader = new SpringFxmlLoader();

 @Override
 public void start(Stage primaryStage) {
  Parent root = (Parent) loader.load(&quot;/search.fxml&quot;);
  Scene scene = new Scene(root, 768, 480);
  primaryStage.setScene(scene);
  primaryStage.setTitle(&quot;JavaFX demo&quot;);
  primaryStage.show();
 }

 public static void main(String[] args) {
  launch(args);
 }
}
</pre> 

The only special thing to note is that we extend from Application. This is a bit boiler plate code which will for example make sure that creating of the UI happens on the JavaFX application thread. You might remember such stories from Swing, where every UI interaction needs to occur on the event dispatcher thread (EDT), this is the same with JavaFX. You are by default on the “right thread” when you are called back by the application (in for example action listeners alike methods). But if you start the application or perform long running tasks in separate threads you need to make sure you start UI interaction on the right thread. For swing you would use <a target="_blank" href="http://docs.oracle.com/javase/7/docs/api/javax/swing/SwingUtilities.html#invokeLater%28java.lang.Runnable%29">SwingUtilities.invokeLater()</a> for JavaFX: <a target="_blank" href="http://docs.oracle.com/javafx/2/api/javafx/application/Platform.html">Platform.runLater()</a>.
  More special is our SpringFxmlLoader:

<pre class="brush: java;highlight: [22,23,24,25,26,27]">
package be.error.javafx;

import java.io.IOException;
import java.io.InputStream;

import javafx.fxml.FXMLLoader;
import javafx.util.Callback;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SpringFxmlLoader {

 private static final ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringApplicationConfig.class);

 public Object load(String url) {
  try (InputStream fxmlStream = SpringFxmlLoader.class
    .getResourceAsStream(url)) {
   System.err.println(SpringFxmlLoader.class
    .getResourceAsStream(url));
   FXMLLoader loader = new FXMLLoader();
   loader.setControllerFactory(new Callback&lt;Class&lt;?&gt;, Object&gt;() {
    @Override
    public Object call(Class&lt;?&gt; clazz) {
     return applicationContext.getBean(clazz);
    }
   });
   return loader.load(fxmlStream);
  } catch (IOException ioException) {
   throw new RuntimeException(ioException);
  }
 }
}
</pre>

The highlighted lines show the custom ControllerFactory. Without setting this JavaFX will simply instantiate the class you specified as controller in the FXML without anything special. In that case the class will not be Spring managed (unless you would be using CTW/LTW AOP). By specifying a custom factory we can define how the controller should be instantiated. In this case we lookup the bean from the application context.  Finally we have our two controllers, the SearchController:  

<pre class="brush: java; highlight: [24, 26, 38, 42, 54]">
package be.error.javafx.controller;

import java.math.BigDecimal;
import java.net.URL;
import java.util.ResourceBundle;

import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.Button;
import javafx.scene.control.TableView;
import javafx.scene.control.TextField;

import org.apache.commons.lang.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;

import be.error.javafx.model.Order;
import be.error.javafx.model.OrderSearchCriteria;
import be.error.javafx.model.OrderService;

public class SearchController implements Initializable {

 @Autowired
 private OrderService orderService;
 @FXML
 private Button search;
 @FXML
 private TableView&lt;Order&gt; table;
 @FXML
 private TextField productName;
 @FXML
 private TextField minPrice;
 @FXML
 private TextField maxPrice;

 @Override
 public void initialize(URL location, ResourceBundle resources) {
  table.setColumnResizePolicy(TableView.CONSTRAINED_RESIZE_POLICY);
 }

 public void search() {
  OrderSearchCriteria orderSearchCriteria = new OrderSearchCriteria();
  orderSearchCriteria.setProductName(productName.getText());
  orderSearchCriteria
    .setMaxPrice(StringUtils.isEmpty(minPrice.getText()) ? null:new BigDecimal(minPrice.getText()));
  orderSearchCriteria
    .setMinPrice(StringUtils.isEmpty(minPrice.getText()) ? null: new BigDecimal(minPrice.getText()));
  ObservableList&lt;Order&gt; rows = FXCollections.observableArrayList();
  rows.addAll(orderService.findOrders(orderSearchCriteria));
  table.setItems(rows);
 }

 public void clear() {
  table.setItems(null);
  productName.setText(&quot;&quot;);
  minPrice.setText(&quot;&quot;);
  maxPrice.setText(&quot;&quot;);
 }
}
</pre> The highlighted lines in respective order:

<ul>
	<li>Auto injection by Spring, this is our Spring managed service which we will use to lookup data from</li>
	<li>Auto injection by JavaFX, our controls that we need to manipulate or read from in our controller</li>
	<li>Special init method to initialize our table so columns will auto resize when the view is enlarged</li>
	<li>action listener style callback which is invoked when the search button is pressed</li>
	<li>action listener style callback which is invoked when the clear button is pressed</li>
</ul>

Finally the FileMenuController which does nothing special besides closing our app:  

<pre class="brush: java;">
package be.error.javafx.controller;

import javafx.application.Platform;
import javafx.event.ActionEvent;

public class FileMenuController {

 public void exit(ActionEvent actionEvent) {
  Platform.exit();
 }
}
</pre> 

And finally the (not so exciting) result:  

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_1-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_1-small.png"/></a></div>

After searching:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_2-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_2-small.png"/></a></div>

Making view wider, also stretches the columns:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_3-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_3-small.png"/></a></div>

The file menu allowing us the exit:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_4-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2012-11-19-javafx-2-with-spring/javafx_4-small.png"/></a></div>

After playing a bit with JavaFX2 I was pretty impressed. There are also more and more controls coming (I believe there is already a browser control and such). So I think we are on the right track here.
