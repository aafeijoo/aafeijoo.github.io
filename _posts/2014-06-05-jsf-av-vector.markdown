---
layout: post
title: "JSF AV C++ - Simulating a Vector using the fixed size Array class"
date: 2014-06-05 11:40:04
categories: source code
tags: c++ safety critical
---

When developing safety critical systems using the Joint Strike Fighter Air Vehicle C++ Coding Standards it can't be used std::vector or most of the other STL collection classes, because they rely on dynamic memory allocation.

However, the behavior of a vector can be simulated using the fixed size Array class, as is shown below.

{% highlight c++ %}
#ifndef VECTOR_H
#define VECTOR_H

//----------------------------------------------------------------------------------------------------------------------

#include <Array.h>

//----------------------------------------------------------------------------------------------------------------------

///
/// \class Vector
/// \brief Fixed size vector.
///
/// \tparam T Type of elements.
/// \tparam N Maximum size.
///
template <typename T, uint32 N>
class Vector
    : public Array<T, N>
{

public:

    ///
    /// \brief Vector default constructor.
    ///
    Vector ()
        : m_elements_count(0)
    {
    }

    ///
    /// \brief Gets the number of elements stored in the vector.
    ///
    /// \return Number of elements stored in the vector.
    ///
    uint32 size () const
    {
        return this->m_elements_count;
    }

    ///
    /// \brief Gets the reference to a vector element.
    ///
    /// \param index Position of the element within the vector (from 0 to size() - 1)
    /// \param element Reference to the vector element.
    /// \return True if index is not out of vector bounds, otherwise false.
    ///
    bool at ( uint32 index,
              Reference element )
    {
        bool on_bounds = (index < this->m_elements_count);
        if (on_bounds)
        {
            element = this->m_array[index];
        }
        return on_bounds;
    }

    ///
    /// \brief Returns whether the vector is empty (i.e. whether its size is 0).
    ///
    /// \return True if empty, otherwise false.
    ///
    bool empty () const
    {
        return (this->size() == 0);
    }

    ///
    /// \brief Adds a new element at the end of the vector, after its current last element.
    ///
    /// \param element Reference to the array element.
    /// \return True if success, otherwise false.
    ///
    bool push_back ( const Array<T, N>::Reference element )
    {
        bool on_bounds = this->check_range(this->m_elements_count + 1);
        if (on_bounds)
        {
            this->m_array[this->m_elements_count++] = element;
        }
        return on_bounds;
    }

    ///
    /// \brief Removes the last element in the vector, effectively reducing the container size by one.
    ///
    /// \return True if success, otherwise false.
    ///
    bool pop_back ()
    {
        bool success = !this->empty();
        if (success)
        {
            this->m_elements_count--;
        }
        return success;
    }

protected:

    ///
    /// \brief Number of elements stored in the vector.
    ///
    uint32 m_elements_count;

};

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}
