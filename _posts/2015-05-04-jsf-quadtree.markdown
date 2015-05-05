---
layout: post
title: "JSF AV C++ - Implementation of a Quad Tree"
date: 2015-05-04 15:05:39
categories: source code
tags: c++ safety critical
---

A Quad Tree is a data structure commonly used in GIS to organize spatial data and perform fast searches through them.

The key to be able to develop a Quad Tree following the Joint Strike Fighter Air Vehicle C++ Coding Standards that can be created out of the initialization phase is avoid the usage of dynamic memory:

* Constants will determine tree maximum size.
* Node items will not contain data, but pointer to data.
* Usage of a dispatcher of fixed size Array slots.

First it will be shown the implementation of a node item, each consisting of an area and a pointer to data.

{% highlight c++ %}
#ifndef QUAD_TREE_NODE_ITEM_H
#define QUAD_TREE_NODE_ITEM_H

//----------------------------------------------------------------------------------------------------------------------

#include <Area.h>

//----------------------------------------------------------------------------------------------------------------------

///
/// \class Quad_tree_node_item
/// \brief Implements an item contained within a Quad Tree node.
///
/// \tparam Tree_data_type Type of tree elements.
/// \tparam Area_data_type Type of area boundaries.
///
template <typename Tree_data_type, typename Area_data_type>
class Quad_tree_node_item
{

public:

    ///
    /// \brief Quad_tree_node_item default constructor.
    ///
    Quad_tree_node_item ()
        : m_data(0)
    {
    }

    ///
    /// \brief Quad_tree_node_item full constructor.
    ///
    /// \tparam data Pointer to item data.
    /// \tparam area Item area.
    ///
    Quad_tree_node_item ( Tree_data_type* data,
                          const Area<Area_data_type>& area )
        : m_data(data),
          m_area(area)
    {
    }

    ///
    /// \brief Quad_tree_node_item copy constructor.
    ///
    /// \tparam src Quad_tree_node_item used to construct this one.
    ///
    Quad_tree_node_item ( const Quad_tree_node_item<Tree_data_type, Area_data_type>& src )
        : m_data(src.m_data),
          m_area(src.m_area)
    {
    }

    ///
    /// \brief Quad_tree_node_item destructor.
    ///
    ~Quad_tree_node_item ()
    {
    }

    ///
    /// \brief Quad_tree_node_item assignment operator.
    ///
    /// \tparam src Quad_tree_node_item assigned to this one.
    ///
    Quad_tree_node_item<Tree_data_type, Area_data_type>& operator= (
            const Quad_tree_node_item<Tree_data_type, Area_data_type>& src )
    {
        if (this != &src)
        {
            this->m_data = src.m_data;
            this->m_area = src.m_area;
        }
        return *this;
    }

    ///
    /// \brief Gets item data.
    ///
    /// \return Pointer to item data.
    ///
    const Tree_data_type& get_data () const
    {
        return *(this->m_data);
    }

    ///
    /// \brief Sets pointer to item data.
    ///
    /// \tparam data Pointer to item data.
    ///
    void set_data ( Tree_data_type* data )
    {
        this->m_data = data;
    }

    ///
    /// \brief Gets item area.
    ///
    /// \return Item area.
    ///
    const Area<Area_data_type>& get_area () const
    {
        return this->m_area;
    }

    ///
    /// \brief Sets item area.
    ///
    /// \tparam area Item area.
    ///
    void set_area ( const Area<Area_data_type>& area )
    {
        this->m_area = area;
    }

private:

    ///
    /// \brief Pointer to item data.
    ///
    Tree_data_type* m_data;

    ///
    /// \brief Item area.
    ///
    Area<Area_data_type> m_area;

};

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

Usually, the Quad Tree has:

* A maximum depth which determines the maximum number of nodes (4<sup>maximum depth</sup>).
* A maximum number of items per node.

Flexibility is lost by using constants to set tree size, but it is required to avoid the allocation of dynamic memory.

{% highlight c++ %}
#ifndef QUAD_TREE_CONSTANTS_H
#define QUAD_TREE_CONSTANTS_H

//----------------------------------------------------------------------------------------------------------------------

#include <Universal_types.h>

//----------------------------------------------------------------------------------------------------------------------

///
/// \brief Maximum number of items per node.
///
const uint32 c_max_items_per_node = 10;

///
/// \brief Maximum depth.
///
const uint32 c_max_depth = 5;

///
/// \brief Maximum number of nodes (4 ^ c_max_depth).
///
const uint32 c_max_nodes = 1024;

///
/// \brief Maximum number of items within the Quad Tree.
///
const uint32 c_max_items = c_max_nodes * c_max_items_per_node;

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

The Quad Tree node class implements the core functionality.

{% highlight c++ %}
#ifndef QUAD_TREE_NODE_H
#define QUAD_TREE_NODE_H

//----------------------------------------------------------------------------------------------------------------------

#include <Quad_tree_constants.h>
#include <Quad_tree_node_item.h>
#include <Array_slot_dispatcher.h>
#include <Vector.h>

//----------------------------------------------------------------------------------------------------------------------

///
/// \class Quad_tree_node
/// \brief Implements a Quad Tree node.
///
/// \tparam Tree_data_type Type of tree elements.
/// \tparam Area_data_type Type of area boundaries.
///
template <typename Tree_data_type, typename Area_data_type>
class Quad_tree_node
{

public:

    ///
    /// \brief Types of child of a Quad Tree node.
    ///
    enum Quad_tree_node_child
    {
       bottom_left_child,
       bottom_right_child,
       top_left_child,
       top_right_child
    };

    ///
    /// \brief Quad_tree_node default constructor.
    ///
    Quad_tree_node ()
        : m_parent(0),
          m_bottom_left_child(0),
          m_bottom_right_child(0),
          m_top_left_child(0),
          m_top_right_child(0),
          m_node_depth(0)
    {
    }

    ///
    /// \brief Quad_tree_node constructor.
    ///
    Quad_tree_node ( const Area<Area_data_type>& area )
        : m_area(area),
          m_parent(0),
          m_bottom_left_child(0),
          m_bottom_right_child(0),
          m_top_left_child(0),
          m_top_right_child(0),
          m_node_depth(0)
    {
    }

    ///
    /// \brief Quad_tree_node copy constructor.
    ///
    /// \tparam src Quad_tree_node used to construct this one.
    ///
    Quad_tree_node ( const Quad_tree_node<Tree_data_type, Area_data_type>& src )
        : m_data(src.m_data),
          m_area(src.m_area),
          m_parent(src.m_parent),
          m_bottom_left_child(src.m_bottom_left_child),
          m_bottom_right_child(src.m_bottom_right_child),
          m_top_left_child(src.m_top_left_child),
          m_top_right_child(src.m_top_right_child),
          m_node_depth(src.m_node_depth)
    {
    }

    ///
    /// \brief Quad_tree_node destructor.
    ///
    ~Quad_tree_node ()
    {
    }

    ///
    /// \brief Quad_tree_node assignment operator.
    ///
    /// \tparam src Quad_tree_node assigned to this one.
    ///
    Quad_tree_node<Tree_data_type, Area_data_type>& operator= (
            const Quad_tree_node<Tree_data_type, Area_data_type>& src )
    {
        if (this != &src)
        {
            this->m_data = src.m_data;
            this->m_area = src.m_area;
            this->m_parent = src.m_parent;
            this->m_bottom_left_child = src.m_bottom_left_child;
            this->m_bottom_right_child = src.m_bottom_right_child;
            this->m_top_left_child = src.m_top_left_child;
            this->m_top_right_child = src.m_top_right_child;
            this->m_node_depth = src.m_node_depth;
        }
        return *this;
    }

    ///
    /// \brief Gets node data.
    ///
    /// \return Node data.
    ///
    Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>, c_max_items_per_node> get_data ()
    {
        return this->m_data;
    }

    ///
    /// \brief Gets area occupied by this node.
    ///
    /// \return Area occupied by this node.
    ///
    const Area<Area_data_type>& get_area () const
    {
        return this->m_area;
    }

    ///
    /// \brief Sets area occupied by this node.
    ///
    /// \tparam area Area occupied by this node.
    ///
    void set_area ( const Area<Area_data_type>& area )
    {
        this->m_area = area;
    }

    ///
    /// \brief Gets the parent of this node.
    ///
    /// \return Pointer to the parent of this node.
    ///
    const Quad_tree_node<Tree_data_type, Area_data_type>& get_parent () const
    {
        return *(this->m_parent);
    }

    ///
    /// \brief Gets a child of this node.
    ///
    /// \tparam child Child name.
    /// \return Pointer to child node.
    ///
    const Quad_tree_node<Tree_data_type, Area_data_type>& operator[] ( Quad_tree_node_child child ) const
    {
        Quad_tree_node<Tree_data_type, Area_data_type>* node = 0;
        switch (child)
        {
            case Quad_tree_node::bottom_left_child:
                node = this->m_bottom_left_child;
                break;
            case Quad_tree_node::bottom_right_child:
                node = this->m_bottom_right_child;
                break;
            case Quad_tree_node::top_left_child:
                node = this->m_top_left_child;
                break;
            case Quad_tree_node::top_right_child:
                node = this->m_top_right_child;
                break;
        }
        return *node;
    }

    ///
    /// \brief Sets the parent of this node and calculates node depth.
    ///
    /// \tparam parent Parent of this node.
    ///
    void set_parent ( Quad_tree_node<Tree_data_type, Area_data_type>* parent )
    {
        this->m_parent = parent;
        this->m_node_depth = (parent != 0) ? (parent->get_node_depth() + 1) : 0;
    }

    ///
    /// \brief Gets node depth.
    ///
    /// \return Node depth.
    ///
    uint32 get_node_depth () const
    {
        return this->m_node_depth;
    }

    ///
    /// \brief Function to know if this node is a leaf (it has not any child).
    ///
    /// \return True if leaf node, false otherwise.
    ///
    bool is_leaf () const
    {
        return (this->m_bottom_left_child == 0) &&
               (this->m_bottom_right_child == 0) &&
               (this->m_top_left_child == 0) &&
               (this->m_top_right_child == 0);
    }

    ///
    /// \brief Inserts a new item into this node.
    ///
    /// \tparam item Item to add.
    /// \tparam nodes Nodes dispatcher.
    /// \return True if success, otherwise false (no space for more).
    ///
    bool insert ( const Quad_tree_node_item<Tree_data_type, Area_data_type>& item,
                  Array_slot_dispatcher<Quad_tree_node<Tree_data_type, Area_data_type>, c_max_nodes>& nodes )
    {
        bool success = false;

        if (this->is_leaf())
        {
            if (this->m_data.size() >= c_max_items_per_node)
            {
                if (this->m_node_depth < c_max_depth)
                {
                    success = this->split(nodes) && this->insert_in_children(item, nodes);
                }
                else
                {
                    success = this->m_data.push_back(item);
                }
            }
            else
            {
                success = this->m_data.push_back(item);
            }
        }
        else
        {
            success = this->insert_in_children(item, nodes);
        }

        return success;
    }

    ///
    /// \brief Searches items contained within a concrete area.
    ///
    /// \tparam area Search area.
    /// \return Vector with items found.
    ///
    Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>*, c_max_items> search (
            const Area<Area_data_type>& area )
    {
        Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>*, c_max_items> items;
        this->search(area, items);
        return items;
    }

private:

    ///
    /// \brief Node data: array with node items.
    ///
    Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>, c_max_items_per_node> m_data;

    ///
    /// \brief Area occupied by this node.
    ///
    Area<Area_data_type> m_area;

    ///
    /// \brief Pointer to the parent of this node.
    ///
    Quad_tree_node<Tree_data_type, Area_data_type>* m_parent;

    ///
    /// \brief Bottom-left child of this node.
    ///
    Quad_tree_node<Tree_data_type, Area_data_type>* m_bottom_left_child;

    ///
    /// \brief Bottom-right child of this node.
    ///
    Quad_tree_node<Tree_data_type, Area_data_type>* m_bottom_right_child;

    ///
    /// \brief Top-left child of this node.
    ///
    Quad_tree_node<Tree_data_type, Area_data_type>* m_top_left_child;

    ///
    /// \brief Top-right child of this node.
    ///
    Quad_tree_node<Tree_data_type, Area_data_type>* m_top_right_child;

    ///
    /// \brief Node depth.
    ///
    uint32 m_node_depth;

    ///
    /// \brief Searches items contained within a concrete area.
    ///
    /// \tparam area Search area.
    /// \tparam items Vector to store items found.
    ///
    void search ( const Area<Area_data_type>& area,
                  Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>*, c_max_items>& items )
    {
        // Search into current node
        typename Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>, c_max_items_per_node>::Iterator dataItr;
        for (dataItr = this->m_data.begin(); dataItr != this->m_data.end(); ++dataItr)
        {
            if (area.intersects(dataItr->get_area()))
            {
                items.push_back(dataItr);
            }
        }

        // Search into children
        if (!this->is_leaf())
        {
            if (area.intersects(this->m_bottom_left_child->get_area()))
            {
                this->m_bottom_left_child->search(area, items);
            }
            if (area.intersects(this->m_bottom_right_child->get_area()))
            {
                this->m_bottom_right_child->search(area, items);
            }
            if (area.intersects(this->m_top_left_child->get_area()))
            {
                this->m_top_left_child->search(area, items);
            }
            if (area.intersects(this->m_top_right_child->get_area()))
            {
                this->m_top_right_child->search(area, items);
            }
        }
    }

    ///
    /// \brief Inserts a new item into some child of this node. Its area must be fully contained into the child node in
    ///        order to be added.
    ///
    /// \tparam item Item to add.
    /// \tparam nodes Nodes dispatcher.
    /// \return True if success, otherwise false (no space for more).
    ///
    bool insert_in_children ( const Quad_tree_node_item<Tree_data_type, Area_data_type>& item,
                              Array_slot_dispatcher<Quad_tree_node<Tree_data_type, Area_data_type>, c_max_nodes>& nodes)
    {
        bool success = !this->is_leaf();
        if (success)
        {
            if (this->m_bottom_left_child->get_area().contains(item.get_area()))
            {
                success = this->m_bottom_left_child->insert(item, nodes);
            }
            else if (this->m_bottom_right_child->get_area().contains(item.get_area()))
            {
                success = this->m_bottom_right_child->insert(item, nodes);
            }
            else if (this->m_top_left_child->get_area().contains(item.get_area()))
            {
                success = this->m_top_left_child->insert(item, nodes);
            }
            else if (this->m_top_right_child->get_area().contains(item.get_area()))
            {
                success = this->m_top_right_child->insert(item, nodes);
            }
            else
            {
                success = this->m_data.push_back(item);
            }
        }
        return success;
    }

    ///
    /// \brief Splits this node into 4.
    ///
    /// \tparam nodes Nodes dispatcher.
    /// \return True if success, otherwise false (no space for more).
    ///
    bool split ( Array_slot_dispatcher<Quad_tree_node<Tree_data_type, Area_data_type>, c_max_nodes>& nodes )
    {
        bool success = this->is_leaf() &&
            nodes.dispatch(this->m_bottom_left_child) &&
            nodes.dispatch(this->m_bottom_right_child) &&
            nodes.dispatch(this->m_top_left_child) &&
            nodes.dispatch(this->m_top_right_child);
        if (success)
        {
            // Bottom-left child
            Area<Area_data_type> bottomLeftArea;
            bottomLeftArea.set_left(this->m_area.get_left());
            bottomLeftArea.set_bottom(this->m_area.get_bottom());
            bottomLeftArea.set_right(this->m_area.get_left() + this->m_area.get_width() / 2);
            bottomLeftArea.set_top(this->m_area.get_bottom() + this->m_area.get_height() / 2);
            this->m_bottom_left_child->set_parent(this);

            // Bottom-right child
            Area<Area_data_type> bottomRightArea;
            bottomRightArea.set_left(bottomLeftArea.get_right());
            bottomRightArea.set_bottom(this->m_area.get_bottom());
            bottomRightArea.set_right(this->m_area.get_right());
            bottomRightArea.set_top(bottomLeftArea.get_top());
            this->m_bottom_right_child->set_parent(this);

            // Top-left child
            Area<Area_data_type> topLeftArea;
            topLeftArea.set_left(this->m_area.get_left());
            topLeftArea.set_bottom(bottomLeftArea.get_top());
            topLeftArea.set_right(bottomLeftArea.get_right());
            topLeftArea.set_top(this->m_area.get_top());
            this->m_top_left_child->set_parent(this);

            // Top-right child
            Area<Area_data_type> topRightArea;
            topRightArea.set_left(bottomLeftArea.get_right());
            topRightArea.set_bottom(bottomLeftArea.get_top());
            topRightArea.set_right(this->m_area.get_right());
            topRightArea.set_top(this->m_area.get_top());
            this->m_top_right_child->set_parent(this);

            // Insert items of this node to its new children
            typename Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>, c_max_items_per_node>::Iterator dataItr;
            for (dataItr = this->m_data.begin(); dataItr != this->m_data.end(); ++dataItr)
            {
                this->insert_in_children(*dataItr, nodes);
            }
            this->m_data.clear();
        }
        return success;
    }

};

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}

Finally, the Quad Tree class will contain only the root node and the dispatcher of Array slots with nodes.

{% highlight c++ %}
#ifndef QUAD_TREE_H
#define QUAD_TREE_H

//----------------------------------------------------------------------------------------------------------------------

#include <Quad_tree_node.h>

//----------------------------------------------------------------------------------------------------------------------

///
/// \class Quad_tree
/// \brief Implements a Quad Tree data structure.
///
/// \tparam Tree_data_type Type of tree elements.
/// \tparam Area_data_type Type of area boundaries.
///
template <typename Tree_data_type, typename Area_data_type>
class Quad_tree
{

public:

    ///
    /// \brief Quad_tree constructor.
    ///
    /// \tparam area Area covered by the tree.
    ///
    Quad_tree ( const Area<Area_data_type>& area )
    {
        this->m_root.set_area(area);
    }

    ///
    /// \brief Quad_tree destructor.
    ///
    ~Quad_tree ()
    {
    }

    ///
    /// \brief Inserts a new item into the tree.
    ///
    /// \tparam data Pointer to item data.
    /// \tparam area Item area.
    /// \return True if success, otherwise false (no space for more).
    ///
    bool insert ( Tree_data_type* data,
                  const Area<Area_data_type>& area )
    {
        Quad_tree_node_item<Tree_data_type, Area_data_type> item(data, area);
        return this->m_root.insert(item, this->m_node_dispatcher);
    }

    ///
    /// \brief Searches items contained within a concrete area.
    ///
    /// \tparam area Search area.
    /// \return Vector with items found.
    ///
    Vector<Quad_tree_node_item<Tree_data_type, Area_data_type>*, c_max_items> search (
            const Area<Area_data_type>& area )
    {
        return this->m_root.search(area);
    }

private:

    ///
    /// \brief Tree root.
    ///
    Quad_tree_node<Tree_data_type, Area_data_type> m_root;

    ///
    /// \brief Node dispatcher.
    ///
    Array_slot_dispatcher<Quad_tree_node<Tree_data_type, Area_data_type>, c_max_nodes> m_node_dispatcher;

    ///
    /// \brief Private Quad_tree copy constructor to avoid its usage.
    ///
    /// \tparam src Quad_tree used to construct this one.
    ///
    Quad_tree ( const Quad_tree<Tree_data_type, Area_data_type>& src )
    {
    }

    ///
    /// \brief Private Quad_tree assignment operator to avoid its usage.
    ///
    /// \tparam src Quad_tree assigned to this one.
    ///
    Quad_tree<Tree_data_type, Area_data_type>& operator= ( const Quad_tree<Tree_data_type, Area_data_type>& src )
    {
        return *this;
    }

};

//----------------------------------------------------------------------------------------------------------------------

#endif
{% endhighlight %}
