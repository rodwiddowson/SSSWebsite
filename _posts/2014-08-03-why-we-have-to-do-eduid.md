---
layout: post
title:  "Why we have to do EduId"
date:   2014-08-03 13:13:13
---

As most of you know, as well as doing Windows file systems I am also a
Shibboleth developer.  This post is provoked by Nicole's excellent post.

As background, when I first got involved with Shibboleth it appear to both
myself and Ian that Discovery was going to need some work, however discovery
has always been if not the unloved, certainly the neglected child. This is
really due to two things:

1.	Security people do not do good GUIs --- as for myself, most of my background is in developing kernel-mode file systems.
2.	Given a choice of making a GUI better of fixing real security issues, which are you going to choose.

This is why I was delighted when the Cardiff stuff on SP usability came out. At the same time JANET (who run the UK Federation on behalf of JISC) commissioned some work to improve the UK Federation WAYF.

At last Discovery is getting the attention I have long thought it deserved and focus is being brought by those with appropriate skills. I cannot emphasize how much I welcome this.

So what is EduId? I cannot speak authoritatively as to what it exactly is, but the way I think of it is as a standard logo that people can look for on a web page. When they see that they can tell that if they click on it they will get to a standardised discovery and at the end their IdP.

A weak, but compelling analogy is with the VISA logo. When you see that sign on the door of a shop you know that you will be embarking on a purchasing experience you understand. You may not get there since your credit might be bad, or there may be geographical limitations on your card.

Similarly if you see the EduId logo you will know that the next stage it to enter your IdP. I believe that having such a logo will definitely help the login experience. No more hunting for "Shibboleth Login" or "SAML Login" or "Login" or whatever. The Cardiff study showed us –-- the experience right now is appalling, and we just have to make it better.
 
Not everyone is enthused however, and this post is my attempt to answer some of the criticisms Comments I have heard are:

1. **What’s wrong with Login?** Well, it’s an overused word to start with. "Log on to the BBC website" they say when they mean "Browse to". Also Login is heavily used by people with local logins. Of course these people should be doing discovery as well, but that’s another story.
2. **It doesn’t solve the issue outside Education.** True, but Education is where the issue is and not solving it here is like saying that you are not going to search for peace in because it doesn’t solve world peace.
3. **It doesn’t solve the issue outside Education and may make it worse when the rest of world starts having this issue.** I have a sneaking suspicion that this might be the case, but
    1. It is not obvious to me that the issue outside education is technical, rather if there is one, it’s a business issue. Either the Commericals IdPs care about owning your identity in which case they will pull every trick they can to make sure that they are the only listed IdP (in which case education can go whistle) or they don’t in which case they do not have a problem or they can align with eduID
    2. I think the level of brokenness we currently have trumps any potential for future brokenness.
    3. It is not obvious to me that logo need not lead to commercial IdPs. After all (to return to my analogy) if I see a VISA logo I might well decide to try my master card (or even my Amex).
4. **The institution is the brand, not eduID.** Well sort of. To go back to the VISA analogy, I follow the VISA sign and that is neutral to the bank that issued my card. The protocol that goes on afterwards is specific to the bank. This is however pushing the analogy too far. Yes, the important brand is the institution, but without ending up as Nascar pages --– which will really mean the huge institutional IdPs and no one else –-- we need something.

Of course what we really want to do is to reclaim the word login. But I fear that that boat has sailed.

As a final note.  This website isn't setup to do blogs very well, I'll be
fixing that, but meantime you have any comments, feel free to contact me at
rdw at steadingsoftware dot com.
