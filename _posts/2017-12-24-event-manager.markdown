---
layout: post
title: "Implementation of a custom Event System"
date: 2017-12-24 20:24:11
categories: source code
tags: c++
---

Sometimes we need to communicate changes between independent parts of our application. The implementation of an event system will help us getting the job done.

The next event system is based on the Observer pattern. The particularity of this implementation is that the events are handled on the same thread where they were raised. The thread that fires an event stops until all its listeners execute their handler code. Good news are that we don't have to allocate/deallocate new dynamic memory for event arguments.

First of all, we have to define the types of events that our system is going to handle.

{% highlight c++ %}
#ifndef EVENT_TYPE_H
#define EVENT_TYPE_H

//----------------------------------------------------------------------------------------------------------------------

namespace Event
{

//----------------------------------------------------------------------------------------------------------------------

    // Types of events
    enum Event_type
    {
        event_whatever_you_want/*,
        ...
        */
    };

//----------------------------------------------------------------------------------------------------------------------

}

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

When an event is fired, it is usually interesting to pass some arguments associated to it. To be able to do this, we will use the next two structures.

{% highlight c++ %}
#ifndef EVENT_ARGUMENT_H
#define EVENT_ARGUMENT_H

//----------------------------------------------------------------------------------------------------------------------

#include <Universal_types.h>
#include <string>
#include <map>

//----------------------------------------------------------------------------------------------------------------------

namespace Event
{

//----------------------------------------------------------------------------------------------------------------------

    // Structure to store information about generic event arguments
    struct Event_property
    {
        // Property identifier
        int32 id;

        // Type of property
        Universal_types type;

        // Value of property
        const void* value;

        //
        // @brief Event_property default constructor.
        //
        Event_property ()
            : id(0),
              value(0)
        {
        }

        //
        // @brief Event_property full constructor.
        //
        // @param id Property identifier.
        // @param type Type of property.
        // @param value Value of property.
        //
        Event_property ( int32 id,
                         Universal_types type,
                         const void* value )
            : id(id),
              type(type),
              value(value)
        {
        }

    };

    // Structure to store information about generic event arguments
    struct Event_argument
    {
        // Argument identifier
        int32 id;

        // Argument properties
        std::map<int32, Event_property> properties;

        //
        // @brief Event_argument default constructor.
        //
        Event_argument ()
            : id(0)
        {
        }

        //
        // @brief Event_argument constructor.
        //
        // @param id Argument identifier.
        //
        Event_argument ( int32 id )
            : id(id)
        {
        }

        //
        // @brief Event_argument full constructor.
        //
        // @param id Argument identifier.
        // @param properties Argument properties.
        //
        Event_argument ( int32 id,
                         const std::map<int32, Event_property>& properties )
            : id(id),
              properties(properties)
        {
        }

    };

//----------------------------------------------------------------------------------------------------------------------

}

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

You should notice the Universal_types enumeration. It is a custom way to define the type of an event argument.

{% highlight c++ %}
#ifndef UNIVERSAL_TYPES_H
#define UNIVERSAL_TYPES_H

//----------------------------------------------------------------------------------------------------------------------

typedef char int8;
typedef short int16;
typedef int int32;
typedef unsigned char uint8;
typedef unsigned short uint16;
typedef unsigned int uint32;
typedef float float32;
typedef double float64;

enum Universal_types
{
    utype_int8,
    utype_int16,
    utype_int32,
    utype_int64,
    utype_uint8,
    utype_uint16,
    utype_uint32,
    utype_uint64,
    utype_float32,
    utype_float64,
    utype_bool,
    utype_string,
    utype_buffer_int8,
    utype_buffer_int16,
    utype_buffer_int32,
    utype_buffer_int64,
    utype_buffer_uint8,
    utype_buffer_uint16,
    utype_buffer_uint32,
    utype_buffer_uint64,
    utype_buffer_float32,
    utype_buffer_float64
};

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

Going on, we have the abstract class Event_listener, that has to be inherited by classes that want to execute some code when a specific event is raised.

{% highlight c++ %}
#ifndef EVENT_LISTENER_H
#define EVENT_LISTENER_H

//----------------------------------------------------------------------------------------------------------------------

#include <Event/Event_type.h>
#include <Event/Event_argument.h>

//----------------------------------------------------------------------------------------------------------------------

namespace Event
{

    class Event_listener
    {

    public:

        // Declaration of friend class
        friend class Event_manager;

        //
        // @brief Event_listener default constructor.
        //
        Event_listener ()
        {
        }

        //
        // @brief Event_listener destructor.
        //
        virtual ~Event_listener ()
        {
        }

    protected:

        //
        // @brief Handles event.
        //
        // @param event_type Type of the event.
        // @param event_argument Argument of the event.
        //
        virtual void handle_event ( Event_type event_type,
                                    const Event_argument& event_argument = Event_argument() ) = 0;

    };

}

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

Finally, the Event_manager is a singleton that will allow classes to subscribe to events and to fire them.
When an event is fired, the event manager iterates all the event listeners associated to the event and calls their handle_event method.

{% highlight c++ %}
#ifndef EVENT_MANAGER_H
#define EVENT_MANAGER_H

//----------------------------------------------------------------------------------------------------------------------

#include <Event/Event_type.h>
#include <Event/Event_argument.h>
#include <Event/Event_listener.h>
#include <map>
#include <vector>

//----------------------------------------------------------------------------------------------------------------------

namespace Event
{

    class Event_manager
    {

    public:

        //
        // @brief Returns a singleton instance of the Event_manager class.
        //
        // @return Event_manager class instance.
        //
        static Event_manager& get_instance ()
        {
            static Event_manager instance;
            return instance;
        }

        //
        // @brief Initializes the Event manager.
        //
        // @return True on success, false otherwise.
        //
        bool init ();

        //
        // @brief Enables the Event manager.
        //
        void enable ();

        //
        // @brief Disables the Event manager.
        //
        void disable ();

        //
        // @brief Subscribes listener to event.
        //
        // @param event_type Type of the event.
        // @param listener Listener.
        //
        void subscribe ( Event_type event_type,
                         Event_listener* listener );

        //
        // @brief Fires event.
        //
        // @param event_type Type of the event.
        // @param event_argument Argument of the event.
        //
        void fire ( Event_type event_type,
                    const Event_argument& event_argument = Event_argument() );

    private:

        //
        // @brief Event_manager default constructor.
        //
        Event_manager ();

        //
        // @brief Disable monitor copy constructor.
        //
        Event_manager ( const Event_manager& src );

        //
        // @brief Event manager destructor.
        //
        ~Event_manager ();

        //
        // @brief Disable Event manager assignment operator.
        //
        void operator= ( const Event_manager& src );

        // Enabled flag
        bool m_enabled;

        // Listeners of each event type
        std::map<Event_type, std::vector<Event_listener*> > m_listeners;

    };

}

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}


{% highlight c++ %}
#include <Event/Event_manager.h>

//----------------------------------------------------------------------------------------------------------------------

namespace Event
{

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Event_manager default constructor.
    //
    Event_manager::Event_manager ()
        : m_enabled(false)
    {
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Event manager destructor.
    //
    Event_manager::~Event_manager ()
    {
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Initializes the Event manager.
    //
    // @return True on success, false otherwise.
    //
    bool Event_manager::init ()
    {
        this->enable();
        return true;
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Enables the Event manager.
    //
    void Event_manager::enable ()
    {
        this->m_enabled = true;
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Disables the Event manager.
    //
    void Event_manager::disable ()
    {
        this->m_enabled = false;
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Subscribes listener to event.
    //
    // @param event_type Type of the event.
    // @param listener Listener.
    //
    void Event_manager::subscribe ( Event_type event_type,
                                    Event_listener* listener )
    {
        this->m_listeners[event_type].push_back(listener);
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Fires event.
    //
    // @param event_type Type of the event.
    // @param event_argument Argument of the event.
    //
    void Event_manager::fire ( Event_type event_type,
                               const Event_argument& event_argument )
    {
        if (this->m_enabled)
        {
            if (this->m_listeners.find(event_type) != this->m_listeners.end())
            {
                std::vector<Event_listener*> listeners = this->m_listeners[event_type];
                for (std::vector<Event_listener*>::iterator itr = listeners.begin(); itr != listeners.end(); ++itr)
                {
                    // REMEMBER:
                    // event_argument.properties contains pointers to values that could be deleted after this call
                    (*itr)->handle_event(event_type, event_argument);
                }
            }
        }
    }

//----------------------------------------------------------------------------------------------------------------------

}

//----------------------------------------------------------------------------------------------------------------------
{% endhighlight %}
