---
layout: post
title: "JSF AV C++ - The Array class"
date: 2014-05-30 11:40:04
categories: source code
tags: c++ safety critical
---

The rule 97 of Joint Strike Fighter Air Vehicle C++ Coding Standards forbids the use of arrays in interfaces.

> Arrays **shall not** be used in interfaces. Instead, the *Array* class should be used.
**Rationale**: Arrays degenerate to pointers when passed as parameters. This "array decay" problem has long been known to be a source of errors.
Note: See Array.doc for guidance concerning the proper use of the *Array* class, including its interaction with memory management and error handling facilities.

Below it is shown a simple implementation of this Array class that wraps a fixed size array that does not need to allocate dynamic memory and handles access attempts out of array bounds.
For full functionality, check out the Array implementation in the C++ TR1.

{% highlight c++ %}
#ifndef ARRAY_H
#define ARRAY_H

//----------------------------------------------------------------------------------------------------------------------

#include <Universal_types.h>

//----------------------------------------------------------------------------------------------------------------------

///
/// \class Array
/// \brief Fixed size array.
///
/// \tparam T Type of elements.
/// \tparam N Number of elements.
///
template <typename T, uint32 N>
class Array
{

public:

    ///
    /// \brief Reference
    ///
    typedef T& Reference;

    ///
    /// \brief Iterator
    ///
    typedef T* Iterator;

    ///
    /// \brief Gets the number of array elements.
    ///
    /// \return Number of array elements.
    ///
    uint32 size () const
    {
        return N;
    }

    ///
    /// \brief Gets the pointer to an array element.
    ///
    /// \param index Position of the element within the array (from 0 to N - 1)
    /// \return Pointer to the array element if index is not out of array bounds, otherwise 0.
    ///
    Iterator at ( uint32 index )
    {
        Iterator itr = 0;
        if (this->check_range(index))
        {
            itr = static_cast<Iterator>(&m_array[index]);
        }
        return itr;
    }

    ///
    /// \brief Gets the pointer to the beginning of the array.
    ///
    /// \return Pointer to the beginning of the array.
    ///
    Iterator begin ()
    {
        return static_cast<Iterator>(&m_array[0]);
    }

    ///
    /// \brief Gets the pointer to the end of the array.
    ///
    /// \return Pointer to the end of the array.
    ///
    Iterator end ()
    {
        return static_cast<Iterator>(&m_array[N]);
    }

    ///
    /// \brief Gets the reference to the first element of the array.
    ///
    /// \return Reference to the first element of the array.
    ///
    Reference front ()
    {
        return *(this->begin());
    }

    ///
    /// \brief Gets the reference to the last element of the array.
    ///
    /// \return Reference to the last element of the array.
    ///
    Reference back ()
    {
        return (N == 0) ? *(this->end()) : *(this->end() - 1);
    }

protected:

    ///
    /// \brief Fixed size array
    ///
    T m_array[N];

    ///
    /// \brief Checks if an index is inside of array range.
    ///
    /// \param index Position of the element within the array (from 0 to N - 1)
    /// \return True if index is not out of array bounds, otherwise false.
    ///
    static bool check_range ( uint32 index )
    {
        return (index < N);
    }

};

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}
