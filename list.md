* list 的节点设计

  ```c++
  template <class _Tp>
  struct _List_node {
    typedef void* _Void_pointer;
    _Void_pointer _M_next;
    _Void_pointer _M_prev;
    _Tp _M_data;
  };
  ```

  是一个双向链表。

* list迭代器的设计

  ```cpp
  template<class _Tp, class _Ref, class _Ptr>
  struct _List_iterator {
    typedef _List_iterator<_Tp,_Tp&,_Tp*>             iterator;
    typedef _List_iterator<_Tp,const _Tp&,const _Tp*> const_iterator;
    typedef _List_iterator<_Tp,_Ref,_Ptr>             _Self;
  
    typedef bidirectional_iterator_tag iterator_category;
    typedef _Tp value_type;
    typedef _Ptr pointer;
    typedef _Ref reference;
    typedef _List_node<_Tp> _Node;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
  
    _Node* _M_node;	// 迭代器内部指针
  
    _List_iterator(_Node* __x) : _M_node(__x) {}
    _List_iterator() {}
    _List_iterator(const iterator& __x) : _M_node(__x._M_node) {}
  
    bool operator==(const _Self& __x) const { return _M_node == __x._M_node; }
    bool operator!=(const _Self& __x) const { return _M_node != __x._M_node; }
    // 这个*运算符重载，是取节点的数据值
    reference operator*() const { return (*_M_node)._M_data; }
  
  #ifndef __SGI_STL_NO_ARROW_OPERATOR
    pointer operator->() const { return &(operator*()); }
  #endif /* __SGI_STL_NO_ARROW_OPERATOR */
  
    // 前置++，指向下一个节点。
    _Self& operator++() { 
      _M_node = (_Node*)(_M_node->_M_next);
      return *this;
    }
    _Self operator++(int) { 
      _Self __tmp = *this;
      ++*this;
      return __tmp;
    }
    _Self& operator--() { 
      _M_node = (_Node*)(_M_node->_M_prev);
      return *this;
    }
    _Self operator--(int) { 
      _Self __tmp = *this;
      --*this;
      return __tmp;
    }
  };
  ```

* list的数据结构

  ```cpp
  template <class _Tp, class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >
  class list : protected _List_base<_Tp, _Alloc> {
    typedef _List_base<_Tp, _Alloc> _Base;
  protected:
    typedef void* _Void_pointer;
  
  public:      
    typedef _Tp value_type;
    typedef value_type* pointer;
    typedef const value_type* const_pointer;
    typedef value_type& reference;
    typedef const value_type& const_reference;
    typedef _List_node<_Tp> _Node;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
  
    typedef typename _Base::allocator_type allocator_type;
    allocator_type get_allocator() const { return _Base::get_allocator(); }
  
  public:
    typedef _List_iterator<_Tp,_Tp&,_Tp*>             iterator;
    typedef _List_iterator<_Tp,const _Tp&,const _Tp*> const_iterator;
  
  #ifdef __STL_CLASS_PARTIAL_SPECIALIZATION
    typedef reverse_iterator<const_iterator> const_reverse_iterator;
    typedef reverse_iterator<iterator>       reverse_iterator;
  #else /* __STL_CLASS_PARTIAL_SPECIALIZATION */
    typedef reverse_bidirectional_iterator<const_iterator,value_type,
                                           const_reference,difference_type>
            const_reverse_iterator;
    typedef reverse_bidirectional_iterator<iterator,value_type,reference,
                                           difference_type>
            reverse_iterator; 
  #endif /* __STL_CLASS_PARTIAL_SPECIALIZATION */
  
  protected:
  #ifdef __STL_HAS_NAMESPACES
    // 仅用一个_M_node就可以表示整个链表，因为是双向的
    using _Base::_M_node;
    using _Base::_M_put_node;
    using _Base::_M_get_node;
  #endif /* __STL_HAS_NAMESPACES */
  
  protected:
    _Node* _M_create_node(const _Tp& __x)
    {
      _Node* __p = _M_get_node();
      __STL_TRY {
        construct(&__p->_M_data, __x);
      }
      __STL_UNWIND(_M_put_node(__p));
      return __p;
    }
  
    _Node* _M_create_node()
    {
      _Node* __p = _M_get_node();
      __STL_TRY {
        construct(&__p->_M_data);
      }
      __STL_UNWIND(_M_put_node(__p));
      return __p;
    }
  
  public:
    explicit list(const allocator_type& __a = allocator_type()) : _Base(__a) {}
  	// _M_node为空，next就是begin
    iterator begin()             { return (_Node*)(_M_node->_M_next); }
    const_iterator begin() const { return (_Node*)(_M_node->_M_next); }
  
    iterator end()             { return _M_node; }
    const_iterator end() const { return _M_node; }
  
    reverse_iterator rbegin() 
      { return reverse_iterator(end()); }
    const_reverse_iterator rbegin() const 
      { return const_reverse_iterator(end()); }
  
    reverse_iterator rend()
      { return reverse_iterator(begin()); }
    const_reverse_iterator rend() const
      { return const_reverse_iterator(begin()); }
  
    bool empty() const { return _M_node->_M_next == _M_node; }
    size_type size() const {
      size_type __result = 0;
      distance(begin(), end(), __result);
      return __result;
    }
    size_type max_size() const { return size_type(-1); }
  
    reference front() { return *begin(); }
    const_reference front() const { return *begin(); }
    reference back() { return *(--end()); }
    const_reference back() const { return *(--end()); }
  
    void swap(list<_Tp, _Alloc>& __x) { __STD::swap(_M_node, __x._M_node); }
  
    iterator insert(iterator __position, const _Tp& __x) {
      _Node* __tmp = _M_create_node(__x);
      __tmp->_M_next = __position._M_node;
      __tmp->_M_prev = __position._M_node->_M_prev;
      ((_Node*) (__position._M_node->_M_prev))->_M_next = __tmp;
      __position._M_node->_M_prev = __tmp;
      return __tmp;
    }
    iterator insert(iterator __position) { return insert(__position, _Tp()); }
  #ifdef __STL_MEMBER_TEMPLATES
    // Check whether it's an integral type.  If so, it's not an iterator.
  
    template<class _Integer>
    void _M_insert_dispatch(iterator __pos, _Integer __n, _Integer __x,
                            __true_type) {
      insert(__pos, (size_type) __n, (_Tp) __x);
    }
  
    template <class _InputIterator>
    void _M_insert_dispatch(iterator __pos,
                            _InputIterator __first, _InputIterator __last,
                            __false_type);
  
    template <class _InputIterator>
    void insert(iterator __pos, _InputIterator __first, _InputIterator __last) {
      typedef typename _Is_integer<_InputIterator>::_Integral _Integral;
      _M_insert_dispatch(__pos, __first, __last, _Integral());
    }
  
  #else /* __STL_MEMBER_TEMPLATES */
    void insert(iterator __position, const _Tp* __first, const _Tp* __last);
    void insert(iterator __position,
                const_iterator __first, const_iterator __last);
  #endif /* __STL_MEMBER_TEMPLATES */
    void insert(iterator __pos, size_type __n, const _Tp& __x);
   
    void push_front(const _Tp& __x) { insert(begin(), __x); }
    void push_front() {insert(begin());}
    void push_back(const _Tp& __x) { insert(end(), __x); }
    void push_back() {insert(end());}
  
    iterator erase(iterator __position) {
      _Node* __next_node = (_Node*) (__position._M_node->_M_next);
      _Node* __prev_node = (_Node*) (__position._M_node->_M_prev);
      __prev_node->_M_next = __next_node;
      __next_node->_M_prev = __prev_node;
      destroy(&__position._M_node->_M_data);
      _M_put_node(__position._M_node);
      return iterator(__next_node);
    }
    iterator erase(iterator __first, iterator __last);
    void clear() { _Base::clear(); }
  
    void resize(size_type __new_size, const _Tp& __x);
    void resize(size_type __new_size) { resize(__new_size, _Tp()); }
  
    void pop_front() { erase(begin()); }
    void pop_back() { 
      iterator __tmp = end();
      erase(--__tmp);
    }
    list(size_type __n, const _Tp& __value,
         const allocator_type& __a = allocator_type())
      : _Base(__a)
      { insert(begin(), __n, __value); }
    explicit list(size_type __n)
      : _Base(allocator_type())
      { insert(begin(), __n, _Tp()); }
  
  #ifdef __STL_MEMBER_TEMPLATES
  
    // We don't need any dispatching tricks here, because insert does all of
    // that anyway.  
    template <class _InputIterator>
    list(_InputIterator __first, _InputIterator __last,
         const allocator_type& __a = allocator_type())
      : _Base(__a)
      { insert(begin(), __first, __last); }
  
  #else /* __STL_MEMBER_TEMPLATES */
  
    list(const _Tp* __first, const _Tp* __last,
         const allocator_type& __a = allocator_type())
      : _Base(__a)
      { insert(begin(), __first, __last); }
    list(const_iterator __first, const_iterator __last,
         const allocator_type& __a = allocator_type())
      : _Base(__a)
      { insert(begin(), __first, __last); }
  
  #endif /* __STL_MEMBER_TEMPLATES */
    list(const list<_Tp, _Alloc>& __x) : _Base(__x.get_allocator())
      { insert(begin(), __x.begin(), __x.end()); }
  
    ~list() { }
  
    list<_Tp, _Alloc>& operator=(const list<_Tp, _Alloc>& __x);
  
  public:
    // assign(), a generalized assignment member function.  Two
    // versions: one that takes a count, and one that takes a range.
    // The range version is a member template, so we dispatch on whether
    // or not the type is an integer.
  
    void assign(size_type __n, const _Tp& __val);
  
  #ifdef __STL_MEMBER_TEMPLATES
  
    template <class _InputIterator>
    void assign(_InputIterator __first, _InputIterator __last) {
      typedef typename _Is_integer<_InputIterator>::_Integral _Integral;
      _M_assign_dispatch(__first, __last, _Integral());
    }
  
    template <class _Integer>
    void _M_assign_dispatch(_Integer __n, _Integer __val, __true_type)
      { assign((size_type) __n, (_Tp) __val); }
  
    template <class _InputIterator>
    void _M_assign_dispatch(_InputIterator __first, _InputIterator __last,
                            __false_type);
  
  #endif /* __STL_MEMBER_TEMPLATES */
  
  protected:
    void transfer(iterator __position, iterator __first, iterator __last) {
      if (__position != __last) {
        // Remove [first, last) from its old position.
        ((_Node*) (__last._M_node->_M_prev))->_M_next     = __position._M_node;
        ((_Node*) (__first._M_node->_M_prev))->_M_next    = __last._M_node;
        ((_Node*) (__position._M_node->_M_prev))->_M_next = __first._M_node; 
  
        // Splice [first, last) into its new position.
        _Node* __tmp = (_Node*) (__position._M_node->_M_prev);
        __position._M_node->_M_prev = __last._M_node->_M_prev;
        __last._M_node->_M_prev      = __first._M_node->_M_prev; 
        __first._M_node->_M_prev    = __tmp;
      }
    }
  
  public:
    void splice(iterator __position, list& __x) {
      if (!__x.empty()) 
        transfer(__position, __x.begin(), __x.end());
    }
    void splice(iterator __position, list&, iterator __i) {
      iterator __j = __i;
      ++__j;
      if (__position == __i || __position == __j) return;
      transfer(__position, __i, __j);
    }
    void splice(iterator __position, list&, iterator __first, iterator __last) {
      if (__first != __last) 
        transfer(__position, __first, __last);
    }
    void remove(const _Tp& __value);
    void unique();
    void merge(list& __x);
    void reverse();
    void sort();
  
  #ifdef __STL_MEMBER_TEMPLATES
    template <class _Predicate> void remove_if(_Predicate);
    template <class _BinaryPredicate> void unique(_BinaryPredicate);
    template <class _StrictWeakOrdering> void merge(list&, _StrictWeakOrdering);
    template <class _StrictWeakOrdering> void sort(_StrictWeakOrdering);
  #endif /* __STL_MEMBER_TEMPLATES */
  
    friend bool operator== __STL_NULL_TMPL_ARGS (
      const list& __x, const list& __y);
  };
  ```

* list的`transfer`接口

  `transfer`接口是`protected`的，公开的是`splice`接口

  测试：

  ```cpp
  void test_splice() {
    int iv[5] = {5,6,7,8,9};
    list<int> ilist1{0, 22, 99, 3, 4};
    list<int> ilist2(iv, iv+5);
    auto ite = find(ilist1.begin(), ilist1.end(), 99);
    ilist1.splice(ite, ilist2);
    for(auto e: ilist1){
      cout << e << " ";
    }
    cout << endl;
  }
  
  //output:
  //0 22 5 6 7 8 9 99 3 4 
  ```

  注：`merge()，reverse()，sort()`都是基于`transfer()`实现的。

* list的`sort`

  list不能使用STL的算法`sort()`，只能使用自己的sort() member function. 因为STL算法的`sort()`只接受RandomAccessIterator

  ```cpp
  template <class _Tp, class _Alloc>
  void list<_Tp, _Alloc>::sort()
  {
    // Do nothing if the list has length 0 or 1.
    if (_M_node->_M_next != _M_node &&
        ((_Node*) (_M_node->_M_next))->_M_next != _M_node) {
      list<_Tp, _Alloc> __carry;
      list<_Tp, _Alloc> __counter[64];
      int __fill = 0;
      while (!empty()) {
        __carry.splice(__carry.begin(), *this, begin());
        int __i = 0;
        while(__i < __fill && !__counter[__i].empty()) {
          __counter[__i].merge(__carry);
          __carry.swap(__counter[__i++]);
        }
        __carry.swap(__counter[__i]);         
        if (__i == __fill) ++__fill;
      } 
  
      for (int __i = 1; __i < __fill; ++__i)
        __counter[__i].merge(__counter[__i-1]);
      swap(__counter[__fill-1]);
    }
  }
  ```

  