# liferay-GET-URL-BY-ARTICLE---SQL

DECLARE @id VARCHAR(16)
DECLARE @counter INT
DECLARE @cnt INT
DECLARE @getid CURSOR

SET @counter= (SELECT COUNT(DISTINCT [articleId]) FROM [LF_STAGING].[dbo].[JournalArticle] WHERE [templateId] LIKE '167436')

SET @getid = CURSOR FOR
SELECT [articleId] FROM [LF_STAGING].[dbo].[JournalArticle] WHERE [templateId] LIKE '167436' GROUP BY [articleId]
SET @cnt = 0



OPEN @getid
FETCH NEXT
FROM @getid INTO @id
WHILE @counter > @cnt
BEGIN

       DECLARE @idb VARCHAR(16)
       DECLARE @cntb INT
       DECLARE @getidb CURSOR
       DECLARE @counterb INT

       SET @getidb = CURSOR FOR
       SELECT [portletId] FROM [LF_STAGING].[dbo].[PortletPreferences] WHERE [preferences] LIKE '%<name>articleId</name><value>'+@id+'</value>%' GROUP BY [portletId]
       SET @cntb = 0

       SET @counterb= (SELECT COUNT(DISTINCT [portletId]) FROM [LF_STAGING].[dbo].[PortletPreferences] WHERE [preferences] LIKE '%<name>articleId</name><value>'+@id+'</value>%')


       OPEN @getidb
       FETCH NEXT
       FROM @getidb INTO @idb
       WHILE @counterb > @cntb
       BEGIN
			DECLARE @counterc INT
			DECLARE @ALLURL CURSOR
			DECLARE @URL VARCHAR(100)
             SET @counterc = (SELECT COUNT(DISTINCT [friendlyURL]) FROM [LF_STAGING].[dbo].[Layout] WHERE [typeSettings] LIKE '%'+@idb+'%')
             IF @counterc > 0 
			 BEGIN
			     SET @ALLURL = CURSOR FOR SELECT [friendlyURL] FROM [LF_STAGING].[dbo].[Layout] WHERE [typeSettings] LIKE '%'+@idb+'%' GROUP BY [friendlyURL]

					   OPEN @ALLURL
					    FETCH NEXT FROM @ALLURL INTO @URL

						WHILE @counterc > 0
						BEGIN
						PRINT @URL

						FETCH NEXT FROM @ALLURL INTO @URL
						SET @counterc = @counterc - 1
						END
						CLOSE @ALLURL
						DEALLOCATE @ALLURL
			END
             
             SET @cntb = @cntb + 1

             FETCH NEXT
             FROM @getidb INTO @idb
       END

       CLOSE @getidb
       DEALLOCATE @getidb

       SET @cnt = @cnt + 1

       FETCH NEXT
       FROM @getid INTO @id
END

CLOSE @getid
DEALLOCATE @getid
