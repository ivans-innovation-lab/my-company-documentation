# Chapter 2 - Organization

Many years ago, Melvin Conway had observed that how organizations were structured would have a strong impact on any systems they created. In his “[How Do Committees Invent](http://www.melconway.com/research/committees.html)" he wrote:

> "Any organization that designs a system \(defined more broadly here than just information systems\) will inevitably produce a design whose structure is a copy of the organization's communication structure."

At the time, the Harvard Business Review rejected his original paper based on the fact that he hadn’t proved his hypothesis. Nonetheless, this observation has become known as Conway’s Law, and the collective experiences of both my colleagues and myself have time and again reinforced the truth of this statement.

When looking to split a large application into parts, often management focuses on the technology layer, leading to UI teams, server-side logic teams, and database teams. When teams are separated along these lines, even simple changes can lead to a cross-team project taking time and budgetary approval.

![](/assets/conways-law.png)

The microservice approach to division is different, splitting up into services organized around **business capability**. Such services take a broad-stack implementation of software for that business area, including user-interface, persistant storage, and any external collaborations. Consequently the teams are cross-functional, including the full range of skills required for the development: user-experience, database, and project management.

![](/assets/PreferFunctionalStaffOrganization.png)

Large monolithic applications can always be modularized around business capabilities too, although that's not the common case. Certainly we would urge a large team building a monolithic application to divide itself along business lines. The main issue we have seen here, is that they tend to be organized around too many contexts. If the monolith spans many of these modular boundaries it can be difficult for individual members of a team to fit them into their short-term memory. Additionally we see that the modular lines require a great deal of discipline to enforce. The necessarily more explicit separation required by service components makes it easier to keep the team boundaries clear.

