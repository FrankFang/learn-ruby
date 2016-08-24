参考：http://www.railstips.org/blog/archives/2006/11/18/class-and-instance-variables-in-ruby/

## 类变量

	class Polygon
		@@sides = 10
		def self.sides
			@@sides
		end
	end

  puts Polygon.sides # => 10

但是继承的时候会被覆盖

	class Triangle < Polygon
		@@sides = 3
	end

	puts Triangle.sides # => 3
	puts Polygon.sides # => 3

## 类的实例变量

类也是一个对象，所以类也可以有它自己的实例变量。

	class Polygon
		@sides = 10
	end

现在查看 Polygon 的类变量和实例变量

	puts Polygon.class_variables # => @@sides
	puts Polygon.instance_variables # => @sides

添加访问器

	class Polygon
		class << self; attr_accessor :sides end
		@sides = 8
	end

	puts Polygon.sides # => 8

子类也可以有自己的 slides 

	class Triangle < Polygon
		@sides = 3
	end

	puts Triangle.sides # => 3
	puts Polygon.sides # => 8

子类也可以有自己的 slides 

	class Triangle < Polygon
		@sides = 3
	end

	puts Triangle.sides # => 3
	puts Polygon.sides # => 8

如何给 slides 设置默认值呢？

用一个 module 专门来处理这个事情

	module ClassLevelInheritableAttributes
		def self.included(base)
			base.extend(ClassMethods)    
		end
		
		module ClassMethods
			def inheritable_attributes(*args)
				@inheritable_attributes ||= [:inheritable_attributes]
				@inheritable_attributes += args
				args.each do |arg|
					class_eval %(
						class << self; attr_accessor :#{arg} end
					)
				end
				@inheritable_attributes
			end
			
			def inherited(subclass)
				@inheritable_attributes.each do |inheritable_attribute|
					instance_var = "@#{inheritable_attribute}"
					subclass.instance_variable_set(instance_var, instance_variable_get(instance_var))
				end
			end
		end
	end

如果一个 class include 了这个 module，就拥有两个类方法：`inheritable_attributes` 和 `inherited`。

原理略

	class Polygon
		include ClassLevelInheritableAttributes
		inheritable_attributes :sides
		@sides = 8
	end

	puts Polygon.sides # => 8

	class Octogon < Polygon; end

	puts Polygon.sides # => 8
	puts Octogon.sides # => 8

更多属性

	class Polygon
		include ClassLevelInheritableAttributes
		inheritable_attributes :sides, :coolness
		@sides    = 8
		@coolness = 'Very'
	end

	class Octogon < Polygon; end

	puts Octogon.coolness # => 'Very'

