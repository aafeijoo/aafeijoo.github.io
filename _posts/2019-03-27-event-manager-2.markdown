---
layout: post
title: "Implementation of a custom Event System (II)"
date: 2019-03-26 22:01:15
categories: source code
tags: c++
---

Some time ago I wrote a [post](http://aafeijoo.github.io/source/code/2017/12/24/event-manager.html) about how to implement a custom Event System using only static memory to save the event arguments.

That implementation is great to avoid using the heap in a controlled enviroment, but it has its limitations. For example, imagine that you have a listener class that is handling an event locking a mutex and the handling code fires an event that finally causes that another event that needs to be handled by the same class is fired... deadlock!

So, if our system is huge, here is how to implement the Event System using dynamic memory. First, we have to modify the Event_property struct to allocate/deallocate its value depending on its type.

{% highlight c++ %}
struct Event_property
{
    // Property identifier
    int32 id;

    // Type of property
    Universal_types type;

    // Value of property
    void* value;

    //
    // @brief Event_property default constructor.
    //
    Event_property ()
        : id(0),
          type(utype_undefined),
          value(0)
    {
    }

    //
    // @brief Event_property full constructor.
    //
    // @param property_id Property identifier.
    // @param property_type Type of property.
    // @param property_value Value of property.
    //
    Event_property ( int32 property_id,
                     Universal_types property_type,
                     void* property_value )
        : id(property_id),
          type(property_type)
    {
        copy_value(property_type, property_value);
    }


    //
    // @brief Event_property copy constructor.
    //
    // @param src Event_property used to construct this one.
    //
    Event_property ( const Event_property& src )
    {
        if (copy_value(src.type, src.value))
        {
            id = src.id;
            type = src.type;
        }
    }

    //
    // @brief Event_property destructor.
    //
    ~Event_property ()
    {
        delete_value();
    }

    //
    // @brief Event_property assignment operator.
    //
    // @param src Event_property assigned to this one.
    // @returns Reference to this Event_property.
    //
    Event_property& operator= ( const Event_property& src )
    {
        if (this != &src)
        {
            delete_value();
            if (copy_value(src.type, src.value))
            {
                id = src.id;
                type = src.type;
            }
        }
        return *this;
    }

    //
    // @brief Deletes data freeing dynamic memory.
    //
    void delete_value ()
    {
        if (value != 0)
        {
            switch (type)
            {
                case utype_int8:
                    delete static_cast<int8*>(value);
                    break;
                case utype_int16:
                    delete static_cast<int16*>(value);
                    break;
                case utype_int32:
                    delete static_cast<int32*>(value);
                    break;
                case utype_int64:
                    delete static_cast<int64*>(value);
                    break;
                case utype_uint8:
                    delete static_cast<uint8*>(value);
                    break;
                case utype_uint16:
                    delete static_cast<uint16*>(value);
                    break;
                case utype_uint32:
                    delete static_cast<uint32*>(value);
                    break;
                case utype_uint64:
                    delete static_cast<uint64*>(value);
                    break;
                case utype_float32:
                    delete static_cast<float32*>(value);
                    break;
                case utype_float64:
                    delete static_cast<float64*>(value);
                    break;
                case utype_bool:
                    delete static_cast<bool*>(value);
                    break;
                case utype_string:
                    delete static_cast<std::string*>(value);
                    break;
            }
            value = 0;
        }
    }

    //
    // @brief Copies the data using dynamic memory.
    //
    // @param property_type Data type.
    // @param property_value Data value.
    // @return True on success, false otherwise.
    //
    bool copy_value ( Universal_types property_type,
                      void* property_value )
    {
        bool success = true;
        if (property_type != utype_undefined)
        {
            int8* value_int8;
            int16* value_int16;
            int32* value_int32;
            int64* value_int64;
            uint8* value_uint8;
            uint16* value_uint16;
            uint32* value_uint32;
            uint64* value_uint64;
            float32* value_float32;
            float64* value_float64;
            bool* value_bool;
            std::string* value_string;

            switch (property_type)
            {
                case utype_int8:
                    value_int8 = new (std::nothrow) int8;
                    if (value_int8 != 0)
                    {
                        *value_int8 = *(reinterpret_cast<int8*>(property_value));
                        value = value_int8;
                    }
                    break;
                case utype_int16:
                    value_int16 = new (std::nothrow) int16;
                    if (value_int16 != 0)
                    {
                        *value_int16 = *(reinterpret_cast<int16*>(property_value));
                        value = value_int16;
                    }
                    break;
                case utype_int32:
                    value_int32 = new (std::nothrow) int32;
                    if (value_int32 != 0)
                    {
                        *value_int32 = *(reinterpret_cast<int32*>(property_value));
                        value = value_int32;
                    }
                    break;
                case utype_int64:
                    value_int64 = new (std::nothrow) int64;
                    if (value_int64 != 0)
                    {
                        *value_int64 = *(reinterpret_cast<int64*>(property_value));
                        value = value_int64;
                    }
                    break;
                case utype_uint8:
                    value_uint8 = new (std::nothrow) uint8;
                    if (value_uint8 != 0)
                    {
                        *value_uint8 = *(reinterpret_cast<uint8*>(property_value));
                        value = value_uint8;
                    }
                    break;
                case utype_uint16:
                    value_uint16 = new (std::nothrow) uint16;
                    if (value_uint16 != 0)
                    {
                        *value_uint16 = *(reinterpret_cast<uint16*>(property_value));
                        value = value_uint16;
                    }
                    break;
                case utype_uint32:
                    value_uint32 = new (std::nothrow) uint32;
                    if (value_uint32 != 0)
                    {
                        *value_uint32 = *(reinterpret_cast<uint32*>(property_value));
                        value = value_uint32;
                    }
                    break;
                case utype_uint64:
                    value_uint64 = new (std::nothrow) uint64;
                    if (value_uint64 != 0)
                    {
                        *value_uint64 = *(reinterpret_cast<uint64*>(property_value));
                        value = value_uint64;
                    }
                    break;
                case utype_float32:
                    value_float32 = new (std::nothrow) float32;
                    if (value_float32 != 0)
                    {
                        *value_float32 = *(reinterpret_cast<float32*>(property_value));
                        value = value_float32;
                    }
                    break;
                case utype_float64:
                    value_float64 = new (std::nothrow) float64;
                    if (value_float64 != 0)
                    {
                        *value_float64 = *(reinterpret_cast<float64*>(property_value));
                        value = value_float64;
                    }
                    break;
                case utype_bool:
                    value_bool = new (std::nothrow) bool;
                    if (value_bool != 0)
                    {
                        *value_bool = *(reinterpret_cast<bool*>(property_value));
                        value = value_bool;
                    }
                    break;
                case utype_string:
                    value_string = new (std::nothrow) std::string;
                    if (value_string != 0)
                    {
                        *value_string = *(reinterpret_cast<std::string*>(property_value));
                        value = value_string;
                    }
                    break;
            }
            success = (value != 0);
        }
        return success;
    }

};
{% endhighlight %}

Now the Event_manager will have a queue with pending events. The handle_event method of the listeners subscribed will be run in a separate thread.

{% highlight c++ %}
#ifndef EVENT_MANAGER_H
#define EVENT_MANAGER_H

//----------------------------------------------------------------------------------------------------------------------

#include <Event/Event_type.h>
#include <Event/Event_argument.h>
#include <Event/Event_listener.h>
#include <map>
#include <vector>
#include <utility>
#include <queue>
#include <pthread.h>

//----------------------------------------------------------------------------------------------------------------------

namespace Event
{

    class Event_manager
    {

    public:

        //
        // @brief Returns a singleton instance of the Event_manager class (without using dynamic memory).
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
        // @brief Starts the work of the Event manager.
        //
        // @return True on success, false otherwise.
        //
        bool start ();

        //
        // @brief Stops the work of the Event manager.
        //
        void stop ();

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
        // @brief Creates the working thread.
        //
        // @param thread_parameters The thread parameters will be an instance of this class.
        // @return Result.
        //
        static void* create_working_thread ( void* thread_parameters );

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

        //
        // @brief Executes the work of the Event manager.
        //
        void run ();

        // Working thread sleep duration in microseconds
        static const int32 c_sleep_duration;

        // Working thread
        pthread_t m_working_thread;

        // Mutex of working thread
        pthread_mutex_t m_working_mutex;

        // Mutex to control the access to the queue
        pthread_mutex_t m_queue_mutex;

        // Working thread running flag
        bool m_working_thread_running;

        // Working thread stopped flag
        bool m_working_thread_stopped;

        // Initialized flag
        bool m_initialized;

        // Enabled flag
        bool m_enabled;

        // Listeners of each event type
        std::map<Event_type, std::vector<Event_listener*> > m_listeners;

        // Queue with events to be fired
        std::queue<std::pair<Event_type, Event_argument> > m_queue;

    };

}

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}


{% highlight c++ %}
#include <Event/Event_manager.h>
#ifdef LINUX
    #include <unistd.h>
#endif

//----------------------------------------------------------------------------------------------------------------------

namespace Event
{

//----------------------------------------------------------------------------------------------------------------------

    // Working thread sleep duration in microseconds
    const int32 Event_manager::c_sleep_duration = 100000;

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Event_manager default constructor.
    //
    Event_manager::Event_manager ()
        : m_working_thread_running(false),
          m_working_thread_stopped(false),
          m_initialized(false),
          m_enabled(false)
    {
        // Initialize mutex of working thread
        pthread_mutexattr_t mutex_attr;
        pthread_mutexattr_init(&mutex_attr);
        pthread_mutexattr_settype(&mutex_attr, PTHREAD_MUTEX_NORMAL);
        pthread_mutex_init(&this->m_working_mutex, &mutex_attr);
        pthread_mutex_init(&this->m_queue_mutex, &mutex_attr);
        pthread_mutexattr_destroy(&mutex_attr);
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Event manager destructor.
    //
    Event_manager::~Event_manager ()
    {
        this->stop();
        pthread_mutex_destroy(&this->m_working_mutex);
        pthread_mutex_destroy(&this->m_queue_mutex);
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Initializes the Event manager.
    //
    // @return True on success, false otherwise.
    //
    bool Event_manager::init ()
    {
        if (!this->m_initialized)
        {
            this->m_enabled = true;
            this->m_initialized = true;
        }
        return this->m_initialized;
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Starts the work of the Event manager.
    //
    // @return True on success, false otherwise.
    //
    bool Event_manager::start ()
    {
        bool success = (this->m_initialized && !this->m_working_thread_running);
        if (success)
        {
            // Create working thread
            pthread_create(&this->m_working_thread, 0, create_working_thread, this);
            pthread_detach(this->m_working_thread);
        }
        return success;
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Stops the work of the Event manager.
    //
    void Event_manager::stop ()
    {
        if (this->m_working_thread_running)
        {
            this->m_working_thread_running = false;
            while (!this->m_working_thread_stopped)
            {
                usleep(1000000);
            }
        }
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Enables the Event manager.
    //
    void Event_manager::enable ()
    {
        pthread_mutex_lock(&this->m_queue_mutex);

        this->m_enabled = true;

        pthread_mutex_unlock(&this->m_queue_mutex);
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Disables the Event manager.
    //
    void Event_manager::disable ()
    {
        pthread_mutex_lock(&this->m_queue_mutex);

        this->m_enabled = false;

        pthread_mutex_unlock(&this->m_queue_mutex);
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
        pthread_mutex_lock(&this->m_working_mutex);

        this->m_listeners[event_type].push_back(listener);

        pthread_mutex_unlock(&this->m_working_mutex);
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
        pthread_mutex_lock(&this->m_queue_mutex);

        if (this->m_enabled)
        {
            this->m_queue.push(std::make_pair(event_type, event_argument));
        }

        pthread_mutex_unlock(&this->m_queue_mutex);
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Creates the working thread.
    //
    // @param thread_parameters The thread parameters will be an instance of this class.
    // @return Result.
    //
    void* Event_manager::create_working_thread ( void* thread_parameters )
    {
        Event_manager* event_manager = reinterpret_cast<Event_manager*>(thread_parameters);
        event_manager->run();
        return 0;
    }

//----------------------------------------------------------------------------------------------------------------------

    //
    // @brief Executes the work of the Event manager.
    //
    void Event_manager::run ()
    {
        this->m_working_thread_running = true;
        while (this->m_working_thread_running)
        {
            bool event_in_queue = false;
            std::pair<Event_type, Event_argument> event;

            pthread_mutex_lock(&this->m_queue_mutex);

            // Get event from queue
            event_in_queue = !this->m_queue.empty();
            if (event_in_queue)
            {
                event = this->m_queue.front();
                this->m_queue.pop();
            }

            pthread_mutex_unlock(&this->m_queue_mutex);

            // Handle event
            if (event_in_queue)
            {
                pthread_mutex_lock(&this->m_working_mutex);

                if (this->m_listeners.find(event.first) != this->m_listeners.end())
                {
                    std::vector<Event_listener*> listeners = this->m_listeners[event.first];
                    for (std::vector<Event_listener*>::iterator itr = listeners.begin(); itr != listeners.end();
                        ++itr)
                    {
                        if ((*itr) != 0)
                        {
                            (*itr)->handle_event(event.first, event.second);
                        }
                    }
                }

                pthread_mutex_unlock(&this->m_working_mutex);
            }

            usleep(c_sleep_duration);
        }

        this->m_working_thread_stopped = true;

        pthread_exit(0);
    }

//----------------------------------------------------------------------------------------------------------------------

}

//----------------------------------------------------------------------------------------------------------------------
{% endhighlight %}
