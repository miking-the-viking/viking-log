 # [Mockery](https://github.com/mockery/mockery)
 
 Mockery is a flexible PHP Mock object framework for unit testing that integrates nicely with PHPUnit (and others). 
 
 If you use Mockery in your unit tests, you can simulate dependencies, specifying how many times what methods should be called, their arguments and return values for very refined unit tests.
 
 ```
 $m = mock();
 $m->shouldReceive("test123")->andReturn(true);
 assertThat($m->test123(), equalTo(true));
 ```
 
 - [Docs](http://docs.mockery.io)
