---
title: STL源码剖析之vector
date: 2018-9-10 21:12:23
categories: 2018年9月
tags: [C++,STL,vector]

---

总结vector的原理


<!-- more -->

## 一、vector的定义

	template<class _Ty,class _Ax>
	class vector:public _Vector_val<_Ty,_Ax>{
	public:
		/******/
	protected:
		pointer _Myfirst;//pointer to beginning of array
		pointer _Mylast;//pointer to current end of sequence
		pointer _Myend;//pointer to end of array
	};

示意图如下：
![image](http://ww1.sinaimg.cn/large/0071ouepgy1fv5pefakcgj30as088my3.jpg)

两个关键大小：
大小：size=_Mylast - _Myfirst；
容量：capacity=_Myend - _Myfirst；
分别对应于resize()、reserve()两个函数。
size表示vector中已有元素的个数，容量表示vector最多可存储的元素的个数；为了降低二次分配时的成本，vector实际配置的大小可能比客户需求的更大一些，以备将来扩充，这就是容量的概念。即capacity>=size，当等于时，容器此时已满，若再要加入新的元素时，就要重新进行内存分配，整个vector的数据都要移动到新内存。二次分配成本较高，在实际操作时，应尽量预留一定空间，避免二次分配。

## 二、构造与析构
### 1、构造
vector的构造函数主要有以下几种：

	vector(): _Mybase(){
		//construct empty vector
		_Buy(0);
	}
	explicit vector(size_type _Count): _Mybase(){
		//construct from _Count * _Ty()
		_Construct_n(_Count, _Ty());
	}
	vector(size_type _Count,const _Ty& _Val): _Mybase(){
		//construct from _Count * _Val
		_Construct_n(_Count,_Val);
	}
	vector(const _Myt& _Right) : _Mybase(_Right._Alval){
		//construct by copying _Right
		if(_Buy(_Right.size()))
			_Mylast=_Ucopy(_Right.begin(),_Right.end(),_Myfirst);
	}

vector优异性能的秘诀之一，就是配置比其所容纳的元素所需更多的内存，一般在使用vector之前，就先预留足够空间，以避免二次分配，这样可以使vector的性能达到最佳。因此元素个数_Count是个远比元素值 _Val重要的参数，因此当构造一个vector时，首要参数一定是元素个数。
由上各构造函数可知，基本上所有构造函数都是基于_Construct _n() 的

	bool _Buy(size_type _Capacity){
		//allocate array with _Capacity elements
		_Myfirst=0,_Mylast=0,_Myend=0;
		if(_Capacity==0)// _Count为0时直接返回
			return (false);
		else{
			//nonempty array,allocate storage
			_Myfirst=this->_Alval.allocate(_Capacity);//分配内存，并更新成员变量
			_Mylast=_Myfirst;
			_Myend=_Myfirst+_Capacity;
		}
		return (true);
	}
	void _Construct_n(size_type _Count,const _Ty& _Val){
		//构造含有_Count个值为_Val的元素的容器
		if(_Buy(_Count))
			_Mylast= _Ufill(_Myfirst,_Count,_Val);
	}
### 2、析构
vector的析构函数很简单，就是先销毁所有已存在的元素，然后释放所有内存

    void _Tidy()
        {   // free all storage
        if (_Myfirst != 0)
            {   // something to free, destroy and deallocate it
            _Destroy(_Myfirst, _Mylast);
            this->_Alval.deallocate(_Myfirst, _Myend - _Myfirst);
            }
        _Myfirst = 0, _Mylast = 0, _Myend = 0;
        }

## 三、插入和删除元素
vector的插入和删除元素是通过push_ back () 、 pop_back()两个接口来实现的，他们的内部实现也非常简单

    void push_back(const _Ty& _Val)
        {   // insert element at end
        if (size() < capacity())
            _Mylast = _Ufill(_Mylast, 1, _Val);
        else
            insert(end(), _Val);    //空间不足时，就会触发内存的二次分配
        }

    void pop_back()
        {   // erase element at end
        if (!empty())
            {   // erase last element
            _Destroy(_Mylast - 1, _Mylast);
            --_Mylast;
            }
        }
当没有备用空间时候，push_back将调用insert进行扩容。将内存扩充原来2倍，然后将原数据复制过来，然后释放掉原数据，最后调整三个迭代器的数值_Myfirst,_Mylast,_Myend。所以扩容之后，指向原vector的迭代器会失效，必须重新调用成员函数获取才可以，所以使用迭代器最好时时刻刻重新获取，否则可能引发错误。


	template <class T, class Alloc>
	void vector<T, Alloc>::insert(iterator position, const T& x) {
	  if (_Mylast != _Myend) {//还有备用空间
	    construct(_Mylast, *(_Mylast - 1));//
	    ++_Mylast;
	    T x_copy = x;
	    copy_backward(position, _Mylast - 2, _Mylast - 1);
	    *position = x_copy;//调整迭代器指向
	  }
	  else {//没有备用空间
	    const size_type old_size = size();
	    const size_type len = old_size != 0 ? 2 * old_size : 1;
	    /*
	        如果原大小为0，则为1，否则扩大为2倍。    
	    */
	    iterator new_start = data_allocator::allocate(len);//分配,并返回首部迭代器
	    iterator new_finish = new_start;
	    __STL_TRY {
	      new_finish = uninitialized_copy(_Myfirst, position, new_start);//将原来数据拷贝到new_start，并返回尾部迭代器
	      construct(new_finish, x);//在尾部构造新元素
	      ++new_finish;//新的尾部加1

	      new_finish = uninitialized_copy(position, _Mylast, new_finish);//这句话没有任何用。
	    }
	    destroy(begin(), end());//析构
	    deallocate();//释放原来vector内存空间

	    //迭代器调整
	    _Myfirst = new_start;
	    _Mylast = new_finish;
	    _Myend = new_start + len;
	  }
	}

## 四、其他接口
### 1、reserve()操作

之前提到过reserve（Count） 函数主要是预留Count大小的空间，对应的是容器的容量，目的是保证（_Myend - _Myfirst）>=Count。只有当空间不足时，才会操作，即重新分配一块内存，将原有元素拷贝到新内存，并销毁原有内存

    void reserve(size_type _Count)
        {   // determine new minimum length of allocated storage
        if (capacity() < _Count)
            {   // not enough room, reallocate
            pointer _Ptr = this->_Alval.allocate(_Count);
            _Umove(begin(), end(), _Ptr);
            size_type _Size = size();
            if (_Myfirst != 0)
                {   // destroy and deallocate old array
                _Destroy(_Myfirst, _Mylast);
                this->_Alval.deallocate(_Myfirst, _Myend - _Myfirst);
                }
            _Myend = _Ptr + _Count;
            _Mylast = _Ptr + _Size;
            _Myfirst = _Ptr;
            }
        }

### 2、resize()操作
resize（Count） 函数主要是用于改变size的，也就是改变vector的大小，最终改变的是（_Mylast - _Myfirst）的值，当size < Count时,就插入元素，当size >Count时，就擦除元素。

    void resize(size_type _Newsize, _Ty _Val)
        {   // determine new length, padding with _Val elements as needed
        if (size() < _Newsize)
            _Insert_n(end(), _Newsize - size(), _Val);
        else if (_Newsize < size())
            erase(begin() + _Newsize, end());
        }

### 3、_Insert_n()操作

resize()操作和insert()操作都会利用到_Insert_n()这个函数，这个函数非常重要，也比其他函数稍微复杂一点
虽然_Insert_n(_where, _Count, _Val ) 函数比较长，但是操作都非常简单，主要可以分为以下几种情况：

1、_Count == 0，不需要插入，直接返回

2、max_size() - size() < _Count，超过系统设置的最大容量，会溢出，造成Xlen（）异常

3、_Capacity < size() + _Count，vector的容量不足以插入Count个元素，需要进行二次分配，扩大vector的容量。 在VS下，vector容量会扩大50%，即 _Capacity = _Capacity + _Capacity / 2;
若仍不足，则 _Capacity = size() + _Count;

	 else if (_Capacity < size() + _Count)
            {   // not enough room, reallocate
            _Capacity = max_size() - _Capacity / 2 < _Capacity
                ? 0 : _Capacity + _Capacity / 2;    // try to grow by 50%
            if (_Capacity < size() + _Count)
                _Capacity = size() + _Count;
            pointer _Newvec = this->_Alval.allocate(_Capacity);
            pointer _Ptr = _Newvec;
            _Ptr = _Umove(_Myfirst, _VEC_ITER_BASE(_Where),_Newvec);    // copy prefix
            _Ptr = _Ufill(_Ptr, _Count, _Val);  // add new stuff
            _Umove(_VEC_ITER_BASE(_Where), _Mylast, _Ptr);  // copy suffix
            //内存释放与变量更新
            }

这种情况下，数据从原始容器移动到新分配内存时是从前到后移动的
这里写图片描述
![image](http://ww1.sinaimg.cn/large/0071ouepgy1fv5tyr1p07j30h20by41b.jpg)
4、空间足够，且被插入元素的位置比较靠近_Mylast,即已有元素的尾部

这种情况下不需要再次进行内存分配，且数据是从后往前操作的。首先是将where~last向后移动，为待插入数据预留Count大小的空间，然后从_Mylast处开始填充，然后将从where处开始填充剩余元素

    else if ((size_type)(_Mylast - _VEC_ITER_BASE(_Where)) < _Count)
            {   // new stuff spills off end
            _Umove(_VEC_ITER_BASE(_Where), _Mylast,
                _VEC_ITER_BASE(_Where) + _Count);   // copy suffix
            _Ufill(_Mylast, _Count - (_Mylast - _VEC_ITER_BASE(_Where)),
                _Val);  // insert new stuff off end
            _Mylast += _Count;
            std::fill(_VEC_ITER_BASE(_Where), _Mylast - _Count,
                _Val);  // insert up to old end
            }

5、空间足够，但插入的位置比较靠前
            {   // new stuff can all be assigned
            _Ty _Tmp = _Val;    // in case _Val is in sequence

            pointer _Oldend = _Mylast;
            _Mylast = _Umove(_Oldend - _Count, _Oldend,
                _Mylast);   // copy suffix
            _STDEXT _Unchecked_move_backward(_VEC_ITER_BASE(_Where), _Oldend - _Count,
                _Oldend);   // copy hole
            std::fill(_VEC_ITER_BASE(_Where), _VEC_ITER_BASE(_Where) + _Count,
                _Tmp);  // insert into hole
            }

### 4、erase()操作

	iterator erase(const_iterator _First_arg,
        const_iterator _Last_arg)
        {   // erase [_First, _Last)
        iterator _First = _Make_iter(_First_arg);
        iterator _Last = _Make_iter(_Last_arg);

        if (_First != _Last)
            {   // worth doing, copy down over hole
            pointer _Ptr = _STDEXT unchecked_copy(_VEC_ITER_BASE(_Last), _Mylast,
                _VEC_ITER_BASE(_First));

            _Destroy(_Ptr, _Mylast);
            _Mylast = _Ptr;
            }
        return (_First);
        }

主要操作就是将后半部分的有效元素向前拷贝，并将后面空间的无效元素析构，并更新_Mylast变量
这里写图片描述
![image](http://ww1.sinaimg.cn/large/0071ouepgy1fv5ua8ciphj30i90ahgns.jpg)
### 5、assign()操作

assign()操作最终都会调用到下面的函数，主要操作是首先擦除容器中已有的全部元素，在从头开始插入Count个Val元素

	void _Assign_n(size_type _Count, const _Ty& _Val)
        {   // assign _Count * _Val
        _Ty _Tmp = _Val;    // in case _Val is in sequence
        erase(begin(), end());
        insert(begin(), _Count, _Tmp);
        }
