---
layout: post
title: "Memory Allocation in safety critical systems"
date: 2015-03-10 09:10:32
categories: source code
tags: c++ safety critical
---

The allocation and deallocation of memory from the heap while the system is running is not a good idea in safety critical systems. All dynamic memory should be allocated during the initialization phase, so we ensure that our system has enough memory during the execution of the critical task.

The rule 206 of Joint Strike Fighter Air Vehicle C++ Coding Standards states:

> Allocation/deallocation from/to the free store (heap) **shall not** occur after initialization.
Note that the "placement" *operator new()*, although not technically dynamic memory, may only be used in low-level memory management routines. See AV Rule 70.1 for object lifetime issues associated with placement *operator new()*. 
**Rationale:** repeated allocation (new/malloc) and deallocation (delete/free) from the free store/heap can result in free store/heap fragmentation and hence non-deterministic delays in free store/heap access. See Alloc.doc for alternatives. 

One good design option to perform this restriction is the use of the heap through singleton managers with their instance stored on the stack, that will allocate/deallocate dynamic memory on their constructors/destructors.

{% highlight c++ %}
#ifndef MY_MANAGER_H
#define MY_MANAGER_H

//----------------------------------------------------------------------------------------------------------------------

class My_data;

//----------------------------------------------------------------------------------------------------------------------

class My_manager
{

public:

    ///
    /// \brief Gets a singleton instance of the My_manager class.
    ///
    /// \return My_manager class instance.
    ///
    static My_manager& get_instance ()
    {
        return ms_instance;
    }

private:

    ///
    /// \brief Constructor.
    ///
    My_manager ();

    ///
    /// \brief Destructor.
    ///
    ~My_manager ();

    ///
    /// \brief Class instance
    ///
    static My_manager ms_instance;
    
    ///
    /// \brief Dynamic data
    ///
    My_data* m_data;

};

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

{% highlight c++ %}
#include <My_manager.h>
#include <My_data.h>

//----------------------------------------------------------------------------------------------------------------------

My_manager My_manager::ms_instance;

//----------------------------------------------------------------------------------------------------------------------

My_manager::My_manager ()
{
    this->m_data = new My_data();
}

//----------------------------------------------------------------------------------------------------------------------

My_manager::~My_manager ()
{
    delete this->m_data;
    this->m_data = 0;
}

//----------------------------------------------------------------------------------------------------------------------
{% endhighlight %}
